- **Liveness probe** — "жив ли контейнер?" Если нет → перезапуск. Без probe K8s знает только что процесс упал, но не что он завис
- **Startup probe** — "запустился ли контейнер?" Пока startup probe не пройдёт, liveness НЕ запускается. Для медленно стартующих приложений
- **Readiness probe** — "готов ли принимать трафик?" Если нет → pod убирается из endpoints Service (Ch.11). Контейнер НЕ перезапускается
- Три типа probe: **httpGet** (HTTP 2xx/3xx = ok), **tcpSocket** (порт открыт = ok), **exec** (exit code 0 = ok)
- **Lifecycle hooks:** postStart (параллельно с main process), preStop (перед SIGTERM). exec или httpGet, НЕ tcpSocket

---

## Liveness Probe

#### "Контейнер ещё жив и работает?"

**Без liveness probe:**
- Приложение зависло (**deadlock**, **infinite loop**, **OOM без crash**)
- K8s **НЕ знает**, что что-то не так
- Pod показывает **Running**, но не отвечает

**С liveness probe:**
- K8s **периодически проверяет** здоровье
- Если probe fails **failureThreshold** раз подряд → **restart container**

^pr-liveness-why

### Конфигурация

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 10    # первая проверка через 10s после старта
  periodSeconds: 5           # проверять каждые 5s
  timeoutSeconds: 2          # ответ должен прийти за 2s
  failureThreshold: 3        # 3 фейла подряд → restart
  successThreshold: 1        # 1 успех → считать здоровым (default)
```

^pr-liveness-config

```
Container started
    │
    ├── 10s (initialDelaySeconds)
    │
    ▼
  Probe ✓ ─── 5s ──→ Probe ✓ ─── 5s ──→ Probe ✗
                                            │
                                         5s ▼
                                        Probe ✗
                                            │
                                         5s ▼
                                        Probe ✗ ← 3 fails (failureThreshold)
                                            │
                                            ▼
                                      RESTART container
```

^pr-liveness-flow

## Startup Probe

#### Проблема

Приложение стартует **2 минуты** (JVM warmup, загрузка данных). Liveness probe с `periodSeconds=5` и `failureThreshold=3` — приложение "не здорово" через **15s** → **restart** → **бесконечный цикл**.

#### Решение: startup probe

- Пока startup probe **НЕ succeeded**, liveness probe **НЕ запускается**
- Startup probe может быть "мягче" (больше **failureThreshold**)
- После success → K8s **переключается на liveness probe**

^pr-startup-why

```yaml
startupProbe:                    # фаза запуска
  httpGet:
    path: /healthz
    port: 8080
  periodSeconds: 10              # проверять каждые 10s
  failureThreshold: 12           # до 120s на запуск
livenessProbe:                   # после запуска
  httpGet:
    path: /healthz
    port: 8080
  periodSeconds: 5               # проверять каждые 5s
  failureThreshold: 2            # 2 фейла → restart (быстро)
```

^pr-startup-config

```
  Startup probe (мягкая)          Liveness probe (строгая)
  ┌─────────────────────┐         ┌──────────────────────┐
  │ period=10s           │  ✓     │ period=5s            │
  │ failure=12           │──────▶ │ failure=2            │
  │ = до 120s на старт   │        │ = быстрая реакция    │
  └─────────────────────┘         └──────────────────────┘
  Fail = норма (ещё не запустился)  Fail = проблема → restart
```

^pr-startup-vs-liveness

## Readiness Probe

#### "Готов ли контейнер принимать ТРАФИК?"

**Отличие от liveness:**

| Probe | При fail |
|---|---|
| **Liveness** | **RESTART** контейнер |
| **Readiness** | **УБРАТЬ** pod из Service endpoints (не получает трафик) |

**Use cases:**
- Приложение **загружает кэш** при старте
- Приложение **временно перегружено**
- Зависимость (**БД**) временно недоступна

> Конфигурация **идентична** liveness probe (`httpGet`/`tcpSocket`/`exec` + thresholds). Подробнее → Ch.11 (Services)

^pr-readiness

## Три типа probe

#### httpGet

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
```

- **HTTP GET** запрос. **2xx/3xx** = success. Всё остальное = fail.
- `/healthz` **НЕ должен** требовать аутентификацию

#### tcpSocket

```yaml
livenessProbe:
  tcpSocket:
    port: 5432
```

- **TCP connect**. Порт открыт = success. Не открыт = fail.
- Для **non-HTTP** приложений (**Redis**, **PostgreSQL**, **gRPC**)

#### exec

```yaml
livenessProbe:
  exec:
    command: ["/bin/healthcheck"]
```

- **Exit code 0** = success. Всё остальное = fail.
- Для приложений **без сети** (batch processing)

> ⚠️ **exec** запускает процесс **ВНУТРИ контейнера** (ресурсы!)

^pr-types

## Best Practices для Probes

**✅ DO:**
- Определяй **liveness probe** для **ВСЕХ** pod'ов
- `/healthz` должен проверять только **ВНУТРЕННЕЕ** здоровье приложения
- **Не проверяй зависимости** (backend, DB) в liveness probe — иначе: backend упал → frontend restart → **каскадный отказ**
- Probe handler должен быть **лёгким** (не жрать CPU/memory)
- Используй **failureThreshold** вместо retry в коде probe handler
- Для **Java**: `httpGet` лучше `exec` (exec запускает JVM)

**❌ DON'T:**
- **НЕ проверяй** connectivity к другим сервисам в liveness
- **НЕ делай** тяжёлые операции в probe handler

^pr-best-practices

## Post-Start Hook

Выполняется **ПАРАЛЛЕЛЬНО** со стартом main process — **НЕ** "после старта", а "при старте".

#### exec

```yaml
lifecycle:
  postStart:
    exec:
      command: ["sh", "-c", "echo started > /tmp/started"]
```

#### httpGet

```yaml
lifecycle:
  postStart:
    httpGet:
      path: /warm-up
      port: 8080
```

#### Поведение

- Пока hook не завершится, container в статусе **Waiting** (ContainerCreating)
- `kubectl logs` и `port-forward` **НЕ работают** пока hook не завершится
- Hook **fail** → **container restart**
- **Блокирует** создание СЛЕДУЮЩЕГО контейнера в pod'е

> ⚠️ **httpGet postStart** к своему же контейнеру → **endless restart loop** (приложение ещё не готово принимать запросы)

^pr-poststart

## Pre-Stop Hook

Выполняется **ПЕРЕД** отправкой **SIGTERM**: `preStop hook → (завершился) → SIGTERM → (grace period) → SIGKILL`

#### exec

```yaml
lifecycle:
  preStop:
    exec:
      command: ["nginx", "-s", "quit"]    # graceful shutdown
```

#### httpGet

```yaml
lifecycle:
  preStop:
    httpGet:
      path: /shutdown
      port: 8080
```

#### sleep

```yaml
lifecycle:
  preStop:
    sleep:
      seconds: 5     # подождать 5s перед SIGTERM
```

#### Поведение

- Hook fail **НЕ предотвращает** termination (контейнер всё равно завершится)
- **Входит в** `terminationGracePeriodSeconds` (не добавляет время сверху)
- Вызывается при **liveness fail**, **pod delete**, **scale down**
- **НЕ вызывается** когда процесс завершился сам

> **Probes ОСТАНАВЛИВАЮТСЯ** когда termination начинается

^pr-prestop

## Полная картина: probe + hooks timeline

```
Container создан
  │
  ├── postStart hook (параллельно с main process)
  │
  ├── startup probe (если определён)
  │     │
  │     ✓ succeeded
  │     │
  │     ├── liveness probe starts
  │     ├── readiness probe starts
  │     │
  │     │   ... нормальная работа ...
  │     │
  │     ├── liveness fails × failureThreshold
  │     │     │
  │     │     ▼
  │     │   termination initiated:
  │     │     preStop hook → SIGTERM → (wait) → SIGKILL
  │     │
  │     ├── readiness fails
  │     │     → pod убирается из Service endpoints
  │     │     → контейнер НЕ перезапускается
  │     │     → readiness succeeds → pod возвращается в endpoints
```

^pr-full-timeline

## Связь
- [[Pod lifecycle]] — фазы pod'а, restart policy, termination sequence (Ch.6)
- [[Pod — что это и зачем]] — pod manifest, container definition (Ch.5)
- [[Service — DNS, endpoints, readiness]] — readiness probe влияет на endpoints (Ch.11)
- [[Deployment — стратегии обновления]] — readiness определяет успешность rollout (Ch.15)
