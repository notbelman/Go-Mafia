- Две механики управляют тем, какой Pod на какую ноду попадёт. **Affinity** — Pod говорит "я хочу туда". **Taints/Tolerations** — нода говорит "я не пускаю к себе". Работают вместе
- **nodeSelector** — простейший способ: Pod запускается только на нодах с указанными labels. Нет подходящей ноды — Pod останется Pending
- **nodeAffinity** — расширенный nodeSelector. Два режима: **required** (строго, как nodeSelector) и **preferred** (желательно, но не обязательно). Плюс операторы In/NotIn/Exists
- **podAffinity** — "запусти меня РЯДОМ с другим Pod'ом" (на той же ноде/зоне). Пример: web рядом с cache для низкой latency
- **podAntiAffinity** — "запусти меня ПОДАЛЬШЕ от другого Pod'а". Главный use case: реплики Deployment'а на РАЗНЫХ нодах для отказоустойчивости
- **Taints** (на ноде) + **Tolerations** (на Pod'е) — нода отталкивает все Pod'ы, кроме тех у кого есть toleration. Пример: GPU-ноды — taint "gpu=true", только Pod'ы с toleration попадут туда. Три эффекта: **NoSchedule** (не ставь новые), **PreferNoSchedule** (постарайся не ставить), **NoExecute** (не ставь новые + выгони существующие)
- Control Plane ноды имеют taint `node-role.kubernetes.io/control-plane: NoSchedule` — поэтому обычные Pod'ы на master не попадают

---

## nodeSelector (простой)

```yaml
spec:
  nodeSelector:
    disktype: ssd                # pod только на ноды с label disktype=ssd
    zone: us-east-1a
```

^aff-nodeselector

## nodeAffinity (расширенный)

```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:     # ОБЯЗАТЕЛЬНО
        nodeSelectorTerms:
        - matchExpressions:
          - key: zone
            operator: In                 # In, NotIn, Exists, DoesNotExist, Gt, Lt
            values: [us-east-1a, us-east-1b]
      preferredDuringSchedulingIgnoredDuringExecution:    # ЖЕЛАТЕЛЬНО
      - weight: 80                       # 1-100, чем больше тем важнее
        preference:
          matchExpressions:
          - key: disktype
            operator: In
            values: [ssd]
```

^aff-node-affinity

```
required  → pod НЕ будет scheduled если нет matching ноды (hard)
preferred → scheduler ПОПРОБУЕТ, но не гарантирует (soft)

IgnoredDuringExecution → если label на ноде изменится ПОСЛЕ scheduling,
                          pod НЕ будет evicted (на будущее планируется
                          RequiredDuringExecution)

Встроенные labels нод:
  kubernetes.io/hostname        → имя ноды
  kubernetes.io/os              → linux / windows
  kubernetes.io/arch            → amd64 / arm64
  topology.kubernetes.io/zone   → зона (us-east-1a)
  topology.kubernetes.io/region → регион (us-east-1)
  node.kubernetes.io/instance-type → тип инстанса (m5.xlarge)
```

^aff-node-labels

## podAffinity и podAntiAffinity

```yaml
spec:
  affinity:
    # podAffinity: размещать РЯДОМ с matching pod'ами
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values: [cache]
        topologyKey: kubernetes.io/hostname    # "на той же ноде"

    # podAntiAffinity: размещать ПОДАЛЬШЕ от matching pod'ов
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values: [kiada]
        topologyKey: kubernetes.io/hostname    # "на РАЗНЫХ нодах"
```

^aff-pod-affinity

```
topologyKey определяет "что значит рядом/далеко":

kubernetes.io/hostname           → та же / разная НОДА
topology.kubernetes.io/zone      → та же / разная ЗОНА
topology.kubernetes.io/region    → тот же / разный РЕГИОН

podAffinity use cases:
  → web pod рядом с cache pod (низкая latency)
  → frontend рядом с backend

podAntiAffinity use cases (САМЫЙ ЧАСТЫЙ ВОПРОС):
  → реплики Deployment'а на РАЗНЫХ нодах
  → реплики БД в разных зонах (HA)
```

^aff-topology-key

## ⭐ Как развернуть реплики на разных нодах

```yaml
# Deployment с podAntiAffinity — реплики на разных нодах:
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kiada
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: kiada
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values: [kiada]
            topologyKey: kubernetes.io/hostname

# ⚠️ required + 3 replicas → нужно минимум 3 ноды!
# Если нод меньше → pod Pending
# Для гибкости: preferred вместо required
```

^aff-replicas-different-nodes

## Taints и Tolerations

```bash
# Добавить taint на ноду:
kubectl taint nodes worker-1 dedicated=gpu:NoSchedule

# Убрать taint:
kubectl taint nodes worker-1 dedicated=gpu:NoSchedule-
```

^taint-commands

```yaml
# Pod с toleration:
spec:
  tolerations:
  - key: dedicated
    operator: Equal          # Equal или Exists
    value: gpu
    effect: NoSchedule       # NoSchedule, PreferNoSchedule, NoExecute
```

^taint-toleration

```
Effects:
  NoSchedule         → новые pod'ы без toleration НЕ scheduled на ноду
                       (существующие pod'ы не трогает)
  PreferNoSchedule   → scheduler ПОПРОБУЕТ избежать, но не гарантирует
  NoExecute          → новые pod'ы не scheduled + СУЩЕСТВУЮЩИЕ pod'ы evicted
                       (можно добавить tolerationSeconds для задержки)

Встроенные taints (K8s добавляет автоматически):
  node.kubernetes.io/not-ready          → нода NotReady (NoExecute)
  node.kubernetes.io/unreachable        → нода unreachable (NoExecute)
  node.kubernetes.io/memory-pressure    → мало памяти (NoSchedule)
  node.kubernetes.io/disk-pressure      → мало диска (NoSchedule)
  node.kubernetes.io/pid-pressure       → мало PID'ов (NoSchedule)
  node.kubernetes.io/unschedulable      → kubectl cordon (NoSchedule)

Control plane node taint:
  node-role.kubernetes.io/control-plane: NoSchedule
  → поэтому обычные pod'ы не попадают на master
```

^taint-effects

## Affinity vs Taints — сравнение

```
                    Affinity / Anti-Affinity      Taints / Tolerations
Кто решает?         Pod выбирает ноду             Нода отталкивает pod'ы
Направление         "хочу туда"                   "не пускаю сюда"
Гранулярность       per-pod spec                  per-node + per-pod
Hard/Soft           required / preferred          NoSchedule / PreferNoSchedule
Eviction            нет (IgnoredDuringExecution)   NoExecute evict'ит pod'ы

Работают ВМЕСТЕ:
  Taint на GPU-нодах → только GPU pod'ы (с toleration) туда попадут
  nodeAffinity → GPU pod'ы ХОТЯТ на GPU-ноды
  → taint отталкивает "чужих", affinity притягивает "своих"
```

^aff-vs-taints

## topologySpreadConstraints (дополнение)

```yaml
# Более гибкая альтернатива podAntiAffinity для распределения:
spec:
  topologySpreadConstraints:
  - maxSkew: 1                              # макс. разница между зонами
    topologyKey: topology.kubernetes.io/zone
    whenUnsatisfiable: DoNotSchedule        # или ScheduleAnyway
    labelSelector:
      matchLabels:
        app: kiada

# "распредели pod'ы равномерно по зонам, разница не больше 1"
# Более выразительно чем podAntiAffinity для multi-zone HA
```

^aff-topology-spread

## Связь

- [[Namespaces, labels, selectors, annotations]] — labels на нодах и pod'ах для affinity (Ch.7)
- [[ReplicaSet]] — pod'ы RS распределяются по нодам (Ch.14)
- [[StatefulSet — концепт]] — node failure + taints = pod Terminating (Ch.16)
- [[Архитектура кластера]] — scheduler использует affinity/taints для placement (Ch.1)
- [[PV, PVC, StorageClass]] — local PV + nodeAffinity (Ch.10)