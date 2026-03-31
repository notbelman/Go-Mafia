- Pod = **группа контейнеров** с shared network и (опционально) shared volumes. Минимальная единица деплоя и скейлинга в K8s
- Контейнеры в pod'е разделяют: **net namespace** (один IP, один port space, один hostname), **IPC namespace**, volumes. НЕ разделяют: filesystem (отдельный mnt namespace у каждого)
- **Один контейнер = один процесс.** Один pod = одно приложение (+ sidecar'ы). Frontend и backend — **разные pod'ы**
- Pod никогда **не растягивается** на несколько нод. Все контейнеры pod'а — на одной ноде
- K8s масштабирует pod'ы целиком (не отдельные контейнеры). Если компоненты масштабируются по-разному → разные pod'ы
- Голый Pod (без Deployment/ReplicaSet) — одноразовый. Умер — никто не пересоздаст. Поэтому в production всегда запускают через Deployment, никогда голый Pod
- **Distroless контейнер** (без shell) нельзя дебажить через `kubectl exec`. Решение: `kubectl debug pod -it --image=netshoot` — добавляет временный debug контейнер к работающему Pod'у с утилитами (curl, tcpdump, dig). Pod не пересоздаётся

---

## Зачем Pod, а не просто контейнер

#### Проблема
- Приложение = несколько **связанных процессов**
- Контейнер = **один процесс** (by design)
- Нужна абстракция для группировки связанных контейнеров

#### Pod решает
- Группирует контейнеры с **shared networking**
- Контейнеры общаются через **localhost** (127.0.0.1)
- **Shared port space:** контейнер A на :8080, контейнер B на :8443
- Могут шарить **volumes** для обмена файлами
- Выглядят как один "виртуальный хост"

## Shared namespaces в Pod

Контейнеры в одном Pod'е:

| Namespace | Shared? | Что это значит |
|---|---|---|
| **net** | ✅ | Один IP, один набор сетевых интерфейсов. Общаются через **localhost**, не могут использовать один порт |
| **UTS** | ✅ | Один **hostname** |
| **IPC** | ✅ | Shared memory, message queues |
| **PID** | ⚙️ | Опционально (`shareProcessNamespace: true`) — общее дерево процессов, видят процессы друг друга |
| **mnt** | ❌ | У каждого **СВОЙ** (отдельная файловая система). Для обмена файлами — shared volume |

```
        Pod (один IP: 10.244.2.4)
    ┌──────────────────────────────┐
    │  Container A    Container B  │
    │  :8080          :8443        │
    │       │              │       │
    │       └──── lo ──────┘       │  ← localhost (127.0.0.1)
    │            eth0              │  ← один внешний IP
    └──────────────────────────────┘
```
^pod-network-diagram

## Когда объединять в один Pod

#### ✅ В один Pod
- Процессы **ДОЛЖНЫ** работать на одном хосте
- Тесно связаны, **дополняют** друг друга
- Масштабируются **ВМЕСТЕ**
- Формируют **единое целое** (не независимые компоненты)

#### ❌ В разные Pod'ы
- Frontend + backend (**разное** масштабирование)
- **Независимые** микросервисы
- Разные lifecycle (один stateless, другой stateful)
- Могут общаться по сети — **нет причины** быть на одном хосте

```
❌ Антипаттерн:                    ✅ Правильно:
┌─────────────────────┐            ┌──────────┐  ┌──────────┐
│ Frontend  Backend   │            │ Frontend │  │ Backend  │
│ container container │            │ container│  │ container│
│       Pod           │            │   Pod    │  │   Pod    │
└─────────────────────┘            └──────────┘  └──────────┘
Нельзя скейлить                    Скейлятся независимо:
по отдельности                     3x frontend, 1x backend
```
^pod-split-diagram

## Pod manifest (минимальный)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kiada
spec:
  containers:
  - name: kiada            # имя контейнера
    image: luksa/kiada:0.1 # образ
    ports:
    - containerPort: 8080  # информативно (не обязательно, но полезно)
```
^pod-manifest

**containerPort** — чисто информационное поле. Не влияет на доступность порта. Но полезно для документации и именования портов. ^pod-containerport

## Взаимодействие с Pod'ом

| Команда | Что делает |
|---|---|
| `kubectl port-forward <pod> 8080:8080` | **Проксирование** на localhost |
| `kubectl logs <pod>` | Логи контейнера (**stdout/stderr**) |
| `kubectl logs <pod> -f` | **Стриминг** логов |
| `kubectl logs <pod> -c <container>` | Логи **конкретного** контейнера |
| `kubectl logs <pod> --previous` | Логи **предыдущего** контейнера (после restart) |
| `kubectl exec -it <pod> -- bash` | **Shell** внутри контейнера |
| `kubectl exec <pod> -c <cont> -- cmd` | Команда в **конкретном** контейнере |
| `kubectl cp <pod>:path localpath` | **Копировать** файл из контейнера |
| `kubectl debug <pod> --image=netshoot` | **Ephemeral** debug container |

## Ephemeral debug containers

Контейнер в production **не содержит** debug-утилит (tcpdump, curl, strace) — нельзя `kubectl exec`, нет нужных бинарников

#### Решение: `kubectl debug`
- Добавляет **временный контейнер** к СУЩЕСТВУЮЩЕМУ pod'у
- **Без пересоздания** pod'а
- Контейнер **удаляется** после завершения

```bash
kubectl debug <pod> -it --image nicolaka/netshoot
# netshoot содержит: tcpdump, curl, dig, strace, ip, ss...
```

> `shareProcessNamespace: true` → debug контейнер видит процессы **всех** контейнеров

## Связь
- [[Multi-container Pods]] — sidecar, init containers, native sidecar (Ch.5)
- [[Probes и Lifecycle Hooks]]] — фазы, conditions, liveness/readiness (Ch.6)
- [[Контейнеры — что внутри]] — namespaces, cgroups, как работает изоляция (Ch.2)
- [[Service — типы и routing]] — как трафик попадает в pod (Ch.11)
- [[Kubernetes API и манифесты]] — структура YAML манифеста (Ch.4)
