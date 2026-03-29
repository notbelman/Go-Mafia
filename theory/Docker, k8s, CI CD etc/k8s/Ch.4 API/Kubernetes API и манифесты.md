- Всё в K8s — **объект в API**. Pods, Deployments, Services, Nodes — всё управляется через REST API (CRUD по HTTP)
- Манифест объекта = 4 секции: **apiVersion/kind** (тип), **metadata** (имя, labels), **spec** (желаемое состояние), **status** (текущее состояние)
- **Ты пишешь spec** → контроллер читает spec, выполняет действия, **пишет status**. Это reconciliation loop
- `kubectl explain <kind>` — встроенная документация по полям. `kubectl explain pod.spec.containers` — drill down
- **Conditions** — список ортогональных состояний объекта (Ready, MemoryPressure, DiskPressure). Каждое: type + status (True/False/Unknown) + reason + message

---

## Всё — объект в API

```
Kubernetes API (REST, HTTP)

  POST   /apis/apps/v1/namespaces/default/deployments    → создать
  GET    /apis/apps/v1/namespaces/default/deployments/x  → прочитать
  PUT    /apis/apps/v1/namespaces/default/deployments/x  → обновить
  DELETE /apis/apps/v1/namespaces/default/deployments/x  → удалить

Всё что существует в кластере = объект в API:
  Pod, Deployment, Service, ConfigMap, Secret,
  Node, PersistentVolume, Ingress, Event...

kubectl — CLI клиент к API Server
Все компоненты K8s (scheduler, controllers, kubelet) тоже работают через API
```
^api-rest

## Структура манифеста (YAML)

```yaml
apiVersion: apps/v1          # ── Type metadata
kind: Deployment              #    какой тип объекта, какая API группа

metadata:                     # ── Object metadata
  name: my-app                #    имя, labels, annotations, namespace
  namespace: default
  labels:
    app: my-app

spec:                         # ── Desired state (ты пишешь)
  replicas: 3                 #    что ты ХОЧЕШЬ
  selector: ...
  template: ...

status:                       # ── Actual state (контроллер пишет)
  replicas: 3                 #    что СЕЙЧАС на самом деле
  conditions: ...
```
^api-manifest

## apiVersion и kind

```
apiVersion = API группа + версия:

  v1                → core группа (Pod, Service, ConfigMap, Node, Secret)
  apps/v1           → Deployment, ReplicaSet, StatefulSet, DaemonSet
  batch/v1          → Job, CronJob
  networking.k8s.io/v1 → Ingress, NetworkPolicy
  gateway.networking.k8s.io/v1 → Gateway, HTTPRoute

kind = тип объекта: Pod, Deployment, Service...

Один объект может быть доступен через несколько API версий
(разные ресурсы → один и тот же объект)
```
^api-version-kind

## Spec vs Status — reconciliation loop

```
         Ты                    Controller               Кластер
          │                        │                        │
          │  пишешь spec           │                        │
          │  (replicas: 3)         │                        │
          ├───────────────────────▶│                        │
          │                        │  читает spec           │
          │                        │  создаёт 3 Pod'а       │
          │                        ├───────────────────────▶│
          │                        │                        │
          │                        │  пишет status           │
          │                        │  (replicas: 3,          │
          │  читаешь status        │   ready: 3)             │
          │◀───────────────────────┤                        │

Не все объекты имеют spec/status:
  Event, ConfigMap, Secret — статические данные, нет контроллера
```
^api-spec-status

## Conditions — состояние объекта

```
status:
  conditions:
  - type: Ready
    status: "True"              # True / False / Unknown
    reason: KubeletReady        # machine-facing (для автоматики)
    message: "kubelet is posting ready status"  # human-facing
    lastTransitionTime: "..."   # когда status изменился
    lastHeartbeatTime: "..."    # последний heartbeat

Node conditions:
  Ready            — нода готова принимать pod'ы
  MemoryPressure   — заканчивается RAM
  DiskPressure     — заканчивается диск
  PIDPressure      — заканчиваются PID'ы

Conditions ортогональны: каждый описывает НЕЗАВИСИМЫЙ аспект состояния
→ лучше чем одно поле "status: healthy/unhealthy"
→ легко расширять новыми conditions
```
^api-conditions

## Event объекты

```
Event = отдельный объект в API (не часть другого объекта)
  → создаётся контроллерами при действиях/проблемах
  → удаляется через ~1 час (чтобы не нагружать etcd)
  → два типа: Normal и Warning

kubectl get events                        → все события
kubectl get events --field-selector type=Warning  → только проблемы
kubectl describe <kind> <name>            → события этого объекта

Полезно: запускать kubectl get events после каждого изменения
```
^api-events

## kubectl — полезные команды

```
kubectl get <kind>                    → список объектов
kubectl get <kind> <name> -o yaml     → полный YAML манифест
kubectl get <kind> <name> -o json     → JSON формат
kubectl describe <kind> <name>        → human-readable + events + related objects
kubectl explain <kind>                → документация по полям
kubectl explain pod.spec.containers   → drill down в конкретное поле
kubectl explain pods --recursive      → полное дерево полей
kubectl apply -f manifest.yaml        → создать/обновить объект из файла
kubectl delete -f manifest.yaml       → удалить объект
```
^api-kubectl

## Связь
- [[Архитектура кластера]] — API Server как центральный компонент (Ch.1)
- [[Kubernetes — обзор (Kubernetes in Action)]] — декларативная модель: spec = desired state (Ch.1)
- [[Pod — что это и зачем]] — пример объекта с spec/status (Ch.5)
- [[Namespaces, labels, selectors, annotations]] — metadata объекта (Ch.7)
