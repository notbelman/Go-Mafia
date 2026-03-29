- Pod = **группа контейнеров** с shared network и (опционально) shared volumes. Минимальная единица деплоя и скейлинга в K8s
- Контейнеры в pod'е разделяют: **net namespace** (один IP, один port space, один hostname), **IPC namespace**, volumes. НЕ разделяют: filesystem (отдельный mnt namespace у каждого)
- **Один контейнер = один процесс.** Один pod = одно приложение (+ sidecar'ы). Frontend и backend — **разные pod'ы**
- Pod никогда **не растягивается** на несколько нод. Все контейнеры pod'а — на одной ноде
- K8s масштабирует pod'ы целиком (не отдельные контейнеры). Если компоненты масштабируются по-разному → разные pod'ы

---

## Зачем Pod, а не просто контейнер

```
Проблема:
  Приложение = несколько связанных процессов
  Контейнер = один процесс (by design)
  → нужна абстракция для группировки связанных контейнеров

Pod решает:
  → группирует контейнеры с shared networking
  → контейнеры общаются через localhost (127.0.0.1)
  → shared port space: контейнер A на :8080, контейнер B на :8443
  → могут шарить volumes для обмена файлами
  → выглядят как один "виртуальный хост"
```
^pod-why

## Shared namespaces в Pod

```
Контейнеры в одном Pod'е:

  ✅ net namespace  — один IP, один набор сетевых интерфейсов
                     → общаются через localhost
                     → не могут использовать один порт

  ✅ UTS namespace  — один hostname

  ✅ IPC namespace  — shared memory, message queues

  ⚙️ PID namespace  — опционально (shareProcessNamespace: true)
                     → общее дерево процессов, видят процессы друг друга

  ❌ mnt namespace  — у каждого СВОЙ (отдельная файловая система)
                     → для обмена файлами: shared volume
```
^pod-namespaces

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

```
✅ В один Pod:
  → процессы ДОЛЖНЫ работать на одном хосте
  → тесно связаны, дополняют друг друга
  → масштабируются ВМЕСТЕ
  → формируют единое целое (не независимые компоненты)

❌ В разные Pod'ы:
  → frontend + backend (разное масштабирование)
  → независимые микросервисы
  → разные lifecycle (один stateless, другой stateful)
  → могут общаться по сети — нет причины быть на одном хосте
```
^pod-when-split

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

```
kubectl port-forward <pod> 8080:8080  — проксирование на localhost
kubectl logs <pod>                    — логи контейнера (stdout/stderr)
kubectl logs <pod> -f                 — стриминг логов
kubectl logs <pod> -c <container>     — логи конкретного контейнера
kubectl logs <pod> --previous         — логи предыдущего контейнера (после restart)
kubectl exec -it <pod> -- bash        — shell внутри контейнера
kubectl exec <pod> -c <cont> -- cmd   — команда в конкретном контейнере
kubectl cp <pod>:path localpath       — копировать файл из контейнера
kubectl debug <pod> --image=netshoot  — ephemeral debug container
```
^pod-interaction

## Ephemeral debug containers

```
Контейнер в production не содержит debug-утилит (tcpdump, curl, strace)
→ нельзя kubectl exec, нет нужных бинарников

Решение: kubectl debug
  → добавляет временный контейнер к СУЩЕСТВУЮЩЕМУ pod'у
  → без пересоздания pod'а
  → контейнер удаляется после завершения

kubectl debug <pod> -it --image nicolaka/netshoot
  → netshoot содержит: tcpdump, curl, dig, strace, ip, ss...

shareProcessNamespace: true → debug контейнер видит процессы всех контейнеров
```
^pod-debug

## Связь
- [[Multi-container Pods]] — sidecar, init containers, native sidecar (Ch.5)
- [[Pod lifecycle и probes]] — фазы, conditions, liveness/readiness (Ch.6)
- [[Контейнеры — что внутри]] — namespaces, cgroups, как работает изоляция (Ch.2)
- [[Service — типы и routing]] — как трафик попадает в pod (Ch.11)
- [[Kubernetes API и манифесты]] — структура YAML манифеста (Ch.4)
