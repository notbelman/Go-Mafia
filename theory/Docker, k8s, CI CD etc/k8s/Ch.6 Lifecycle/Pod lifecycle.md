- Pod lifecycle = 3 стадии: **Initialization** (init containers последовательно) → **Run** (regular containers параллельно) → **Termination** (graceful shutdown)
- Pod phases: Pending → Running → Succeeded/Failed. Unknown = kubelet не отвечает
- Container states: Waiting → Running → Terminated. Restart = уничтожить старый + создать новый (не "перезапуск")
- **restartPolicy** (на уровне pod'а): Always (default), OnFailure, Never. Exponential backoff: 0→10→20→40→80→160→300s (cap 5 min, reset через 10 min)
- **terminationGracePeriodSeconds** (default 30s): pre-stop hook → SIGTERM → ждём → SIGKILL. Процесс ОБЯЗАН обрабатывать SIGTERM

---

## Три стадии жизни Pod'а

```
┌─────────────────┐  ┌──────────────────────┐  ┌─────────────────┐
│  Initialization  │  │        Run           │  │  Termination    │
│                  │  │                      │  │                 │
│ Init 1 → Init 2 │→ │ [Container A + B]    │→ │ SIGTERM → KILL  │
│ → ... → Init N  │  │  параллельно         │  │ параллельно     │
│  последовательно │  │  + probes + hooks    │  │ для всех cont.  │
└─────────────────┘  └──────────────────────┘  └─────────────────┘
```
^pl-stages

## Pod Phases

```
Pending     → pod создан, ожидает scheduling / pull images / init containers
Running     → хотя бы один контейнер запущен
Succeeded   → все контейнеры завершились с exit code 0 (для Jobs)
Failed      → хотя бы один контейнер завершился с ненулевым exit code
Unknown     → kubelet не отвечает (нода упала / сеть)
```
^pl-phases

## Pod Conditions

```
PodScheduled      → pod назначен на ноду
Initialized       → все init containers завершились успешно
ContainersReady   → все контейнеры ready (необходимо, но не достаточно)
Ready             → pod готов обслуживать клиентов (containers ready + readiness gates)

Каждая condition: type + status (True/False/Unknown) + reason + message
Ready и ContainersReady могут переключаться туда-сюда
PodScheduled и Initialized → True и остаются True
```
^pl-conditions

## Container States

```
Waiting      → ожидает запуска (pull image, waiting for init)
               reason: ContainerCreating, CrashLoopBackOff, ErrImagePull
Running      → процесс(ы) запущены
Terminated   → процесс завершился
               exitCode, reason, startedAt, finishedAt

kubectl get po <name> -o json | jq .status.containerStatuses
```
^pl-container-states

## Restart Policy и Exponential Backoff

```
restartPolicy (в spec pod'а, применяется ко ВСЕМ контейнерам):

  Always     → перезапуск при любом exit code (default)
  OnFailure  → перезапуск только при exit code ≠ 0
  Never      → никогда не перезапускать

Backoff при повторных падениях:
  1-й restart: сразу (0s)
  2-й: 10s
  3-й: 20s
  4-й: 40s
  5-й: 80s
  6-й: 160s
  7+:  300s (cap, 5 min)

  Reset backoff: контейнер проработал 10+ минут без падения
  Статус во время ожидания: CrashLoopBackOff
```
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

```
0     → успешное завершение
1     → ошибка приложения (generic)
137   → 128 + 9 (SIGKILL) — контейнер убит принудительно
143   → 128 + 15 (SIGTERM) — контейнер завершился по SIGTERM
```
^pl-exit-codes

## SIGTERM и Dockerfile

```
❌ Shell form (SIGTERM НЕ доходит до приложения):
   ENTRYPOINT /myapp arg1 arg2
   → shell запускается как PID 1, myapp как child
   → SIGTERM идёт shell'у, shell его НЕ передаёт

✅ Exec form (SIGTERM доходит):
   ENTRYPOINT ["/myapp", "arg1", "arg2"]
   → myapp запускается как PID 1
   → SIGTERM идёт напрямую приложению
```
^pl-sigterm-dockerfile

## imagePullPolicy

```
Always        → всегда проверять registry (default для :latest)
IfNotPresent  → pull только если нет локально (default для конкретных тегов)
Never         → никогда не pull'ить (image должен быть на ноде)

⚠️ Always + registry offline = контейнер НЕ запустится (даже если image есть локально)
```
^pl-image-pull

## Связь
- [[Multi-container Pods]] — init containers, native sidecar, порядок запуска (Ch.5)
- [[Pod lifecycle и probes]] — liveness/startup/readiness probes, hooks (Ch.6)
- [[Pod — что это и зачем]] — что такое pod, манифест (Ch.5)
- [[Deployment — стратегии обновления]] — rolling update учитывает readiness (Ch.15)
