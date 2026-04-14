- Pod lifecycle = 3 стадии: **Initialization** (init containers последовательно) → **Run** (regular containers параллельно) → **Termination** (graceful shutdown)
- Pod phases: Pending → Running → Succeeded/Failed. Unknown = kubelet не отвечает
- Container states: Waiting → Running → Terminated. Restart = уничтожить старый + создать новый (не "перезапуск")
- **restartPolicy** (на уровне pod'а): Always (default), OnFailure, Never. Exponential backoff: 0→10→20→40→80→160→300s (cap 5 min, если контейнер работает 10 min без рестартов — backoff сбрасывается до 0)
- **terminationGracePeriodSeconds** (default 30s): pre-stop hook → SIGTERM → ждём → SIGKILL. Процесс ОБЯЗАН обрабатывать SIGTERM

---

## Три стадии жизни Pod'а

```
┌─────────────────┐  ┌──────────────────────┐  ┌─────────────────┐
│  Initialization │  │        Run           │  │  Termination    │
│                 │  │                      │  │                 │
│ Init 1 → Init 2 │→ │ [Container A + B]    │→ │ SIGTERM → KILL  │
│ → ... → Init N  │  │  параллельно         │  │ параллельно     │
│  последовательно│  │  + probes + hooks    │  │ для всех cont.  │
└─────────────────┘  └──────────────────────┘  └─────────────────┘
```
^pl-stages

## Pod Phases

| Phase | Описание |
|-------|----------|
| **Pending** | Pod создан, ожидает **scheduling** / **pull images** / **init containers** |
| **Running** | Хотя бы один контейнер **запущен** |
| **Succeeded** | Все контейнеры завершились с **exit code 0** (для Jobs) |
| **Failed** | Хотя бы один контейнер завершился с **ненулевым exit code** |
| **Unknown** | **Kubelet** не отвечает (нода упала / сеть) |

^pl-phases

## Pod Conditions

| Condition | Описание |
|-----------|----------|
| **PodScheduled** | Pod назначен на ноду |
| **Initialized** | Все **init containers** завершились успешно |
| **ContainersReady** | Все контейнеры **ready** (необходимо, но не достаточно) |
| **Ready** | Pod готов обслуживать клиентов (**containers ready** + **readiness gates**) |

- Каждая condition: **type** + **status** (`True`/`False`/`Unknown`) + **reason** + **message**
- **Ready** и **ContainersReady** могут переключаться туда-сюда
- **PodScheduled** и **Initialized** → `True` и остаются `True`

^pl-conditions

## Container States

- **Waiting** — ожидает запуска (**pull image**, waiting for init)
	- reason: `ContainerCreating`, `CrashLoopBackOff`, `ErrImagePull`
- **Running** — процесс(ы) **запущены**
- **Terminated** — процесс **завершился**
	- `exitCode`, `reason`, `startedAt`, `finishedAt`

```bash
kubectl get po <name> -o json | jq .status.containerStatuses
```

^pl-container-states

## Restart Policy и Exponential Backoff

#### restartPolicy

> Задаётся в **spec** pod'а, применяется ко **ВСЕМ** контейнерам.

| Policy | Поведение |
|--------|-----------|
| **Always** | Перезапуск при **любом** exit code **(default)** |
| **OnFailure** | Перезапуск только при **exit code ≠ 0** |
| **Never** | **Никогда** не перезапускать |

#### Exponential Backoff при повторных падениях

| Restart | Задержка |
|---------|----------|
| 1-й | **0s** (сразу) |
| 2-й | **10s** |
| 3-й | **20s** |
| 4-й | **40s** |
| 5-й | **80s** |
| 6-й | **160s** |
| 7+ | **300s** (cap, 5 min) |

- **Reset backoff**: контейнер проработал **10+ минут** без падения
- Статус во время ожидания: **CrashLoopBackOff**

^pl-restart-policy

**Restart = не перезапуск, а пересоздание.** Старый контейнер уничтожается, новый создаётся. Данные на filesystem контейнера **теряются** (если нет volume). ^pl-restart-recreate

## Termination Sequence

```
kubectl delete pod <name>

        deletionGracePeriodSeconds (default = terminationGracePeriodSeconds = 30s)
        ├──────────────────────────────────────────────────────────────┤

  ┌──────────┐    ┌──────────┐    ┌──────────┐
  │Container │    │Container │    │Container │
  │    A     │    │    B     │    │    C     │
  │          │    │          │    │          │
  │ preStop  │    │  SIGTERM │    │ preStop  │    ← параллельно для всех
  │  hook    │    │    ↓     │    │  hook    │
  │    ↓     │    │ process  │    │    ↓     │
  │ SIGTERM  │    │ exits    │    │ SIGTERM  │
  │    ↓     │    │          │    │    ↓     │
  │ process  │    │          │    │ SIGKILL  │    ← если не завершился за grace period
  │ exits    │    │          │    │          │
  └──────────┘    └──────────┘    └──────────┘

После всех regular containers → terminate native sidecars (обратный порядок)
```
^pl-termination

## Exit Codes

| Code | Значение |
|------|----------|
| **0** | Успешное завершение |
| **1** | Ошибка приложения (generic) |
| **137** | 128 + 9 (**SIGKILL**) — контейнер убит **принудительно** |
| **143** | 128 + 15 (**SIGTERM**) — контейнер завершился по **SIGTERM** |

^pl-exit-codes

## SIGTERM и Dockerfile

#### Shell form — SIGTERM НЕ доходит до приложения

```dockerfile
ENTRYPOINT /myapp arg1 arg2
```

- **Shell** запускается как **PID 1**, myapp как **child**
- **SIGTERM** идёт shell'у, shell его **НЕ передаёт**

#### Exec form — SIGTERM доходит

```dockerfile
ENTRYPOINT ["/myapp", "arg1", "arg2"]
```

- **myapp** запускается как **PID 1**
- **SIGTERM** идёт **напрямую** приложению

^pl-sigterm-dockerfile

## imagePullPolicy

| Policy | Поведение |
|--------|-----------|
| **Always** | Всегда проверять **registry** (default для `:latest`) |
| **IfNotPresent** | Pull только если **нет локально** (default для конкретных тегов) |
| **Never** | Никогда не pull'ить (image **должен быть на ноде**) |

> **Always** + registry offline = контейнер **НЕ запустится** (даже если image есть локально)

^pl-image-pull

## Связь
- [[Multi-container Pods]] — init containers, native sidecar, порядок запуска (Ch.5)
- [[Probes и Lifecycle Hooks]]] — liveness/startup/readiness probes, hooks (Ch.6)
- [[Pod — что это и зачем]] — что такое pod, манифест (Ch.5)
- [[Deployment — стратегии обновления]] — rolling update учитывает readiness (Ch.15)
