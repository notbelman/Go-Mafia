- **StatefulSet** — для stateful приложений: базы данных (MongoDB, PostgreSQL), очереди (Kafka, RabbitMQ), кэши (Redis Cluster) — всё где каждый инстанс уникален и хранит свои данные. В отличие от Deployment: Pod'ы получают **ordinal имена** (quiz-0, quiz-1...), каждый — свой **PersistentVolumeClaim**, стабильный DNS. 
  Pets vs Cattle: Deployment = cattle (взаимозаменяемы), StatefulSet = pets (уникальная identity)
  **PersistentVolumeClaim**. Pets vs Cattle: Deployment = cattle (взаимозаменяемы), StatefulSet = pets (уникальная identity)
- **Headless Service** (clusterIP: None) + StatefulSet → каждый pod получает DNS record: `quiz-0.quiz-pods.ns.svc.cluster.local`. Stable network identity при пересоздании pod'а
- **volumeClaimTemplates** — PVC создаётся для каждого pod'а автоматически. Scale down → PVC сохраняются (default). Scale up → pod'ы переподключаются к тем же PVC
- **At-most-one semantics** (гарантия что НЕ будет двух pod'ов с одним ordinal одновременно — защита от split-brain)**:** StatefulSet НЕ создаёт replacement pod при node failure (в отличие от ReplicaSet). Нужно ручное удаление `--force --grace-period 0`
- **podManagementPolicy:** OrderedReady (default — по одному, ждёт ready) vs Parallel (все сразу). OrderedReady может привести к deadlock, например: pod-1 ждёт pod-0, но pod-0 не может стать Ready без pod-1 (circular dependency)

---

## StatefulSet vs Deployment

| Параметр | **Deployment** | **StatefulSet** |
|---|---|---|
| **Pod names** | random (hash+suffix) | **ordinal** (quiz-0, quiz-1...) |
| **Pod identity** | fungible (**cattle**) | stable (**pets**) |
| **PVC per pod** | нет (общий PVC) | да (**volumeClaimTemplates**) |
| **Pod creation order** | все сразу | по одному (**OrderedReady**) или сразу |
| **Scale down order** | по правилам | **highest ordinal first** |
| **DNS per pod** | нет | да (через **headless Service**) |
| **Node failure** | auto replacement | **НЕТ** auto replacement (**at-most-one**) |
| **Update strategy** | Recreate / RollingUpdate | RollingUpdate / OnDelete |
| **Owns pods via** | ReplicaSet | **напрямую** (нет RS) |

^sts-vs-deploy

## Manifest

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: quiz
spec:
  serviceName: quiz-pods             # headless Service (обязательно)
  podManagementPolicy: Parallel      # или OrderedReady (default)
  replicas: 3
  selector:
    matchLabels:
      app: quiz
  template:
    metadata:
      labels:
        app: quiz
    spec:
      containers:
      - name: mongo
        image: mongo:5
        volumeMounts:
        - name: db-data
          mountPath: /data/db
      volumes:
      - name: db-data
        persistentVolumeClaim:
          claimName: db-data         # совпадает с volumeClaimTemplates.name
  volumeClaimTemplates:              # PVC template — уникально для StatefulSet
  - metadata:
      name: db-data                  # → PVC: db-data-quiz-0, db-data-quiz-1...
    spec:
      accessModes: [ReadWriteOnce]
      resources:
        requests:
          storage: 1Gi
```

^sts-manifest

## Headless Service для StatefulSet

```yaml
apiVersion: v1
kind: Service
metadata:
  name: quiz-pods                    # governing Service
spec:
  clusterIP: None                    # headless
  publishNotReadyAddresses: true     # DNS records даже для unready pod'ов
  selector:
    app: quiz
  ports:
  - name: mongodb
    port: 27017
```

^sts-headless

#### DNS records

- `quiz-pods.kiada.svc.cluster.local` → **IPs всех pod'ов**
- `quiz-0.quiz-pods.kiada.svc.cluster.local` → **IP quiz-0**
- `quiz-1.quiz-pods.kiada.svc.cluster.local` → **IP quiz-1**
- `quiz-2.quiz-pods.kiada.svc.cluster.local` → **IP quiz-2**

#### SRV records (peer discovery)

- `_mongodb._tcp.quiz-pods.kiada.svc.cluster.local`
- Используется **MongoDB client**: `mongodb+srv://quiz-pods.kiada.svc.cluster.local`

#### Два Service'а — типичный паттерн

| Service | Назначение |
|---|---|
| **quiz-pods** (headless) | **peer discovery**, `publishNotReadyAddresses: true` |
| **quiz** (regular) | **client traffic**, только **ready** pod'ы |

^sts-dns

## Pod Names и PVC Names

**StatefulSet: quiz**, replicas: 3

```
Pods:   quiz-0    quiz-1    quiz-2
PVCs:   db-data-quiz-0  db-data-quiz-1  db-data-quiz-2
        ───────┬──────   ──────┬──────
        template name    pod name
```

- **Pod удалён** → новый pod с **тем же именем** → **тот же PVC**
  - **state сохраняется** при пересоздании

#### Labels добавляемые controller'ом

- **controller-revision-hash**: `quiz-7576f64fbc` (как pod-template-hash)
- **statefulset.kubernetes.io/pod-name**: `quiz-0` (для **per-pod Service**)

> **ownerReferences** → StatefulSet **напрямую** (не через ReplicaSet)

^sts-naming

## Scaling

#### Scale up

- Новые **pod'ы** + новые **PVC** создаются
- **Ordinal numbers** продолжаются (quiz-3, quiz-4...)

#### Scale down

- Pod с **наибольшим ordinal** удаляется первым
- PVC по умолчанию **сохраняются** (**Retain**)
- Scale up обратно → pod **переподключается** к тому же PVC

#### Scale down to 0

- Все pod'ы удалены, **PVC остаются**
- Scale up → все PVC **переподключаются**

> **Stateful приложения** могут требовать доп. конфигурации при scaling (например, **MongoDB replica set reconfiguration**)

^sts-scaling

## PVC Retention Policy

```yaml
spec:
  persistentVolumeClaimRetentionPolicy:
    whenScaled: Retain      # default: Retain. При scale down
    whenDeleted: Retain      # default: Retain. При удалении StatefulSet
```

- **whenScaled: Delete** → PVC удаляются при **scale down**
- **whenDeleted: Delete** → PVC удаляются при **удалении StatefulSet**

> **whenScaled: Delete** + scale to 0 → **все данные потеряны**
> Лучше: **Retain** + ручное удаление PVC

^sts-pvc-retention

## Node Failure — At-Most-One Semantics

#### ReplicaSet при node failure

- Через несколько минут создаёт **replacement pod** на другой ноде
- Возможно **два экземпляра** с одинаковыми данными работают одновременно

#### StatefulSet при node failure

- Pod отмечается как **Terminating**
- **НЕ создаёт replacement** автоматически
- Причина: **at-most-one guarantee** (два pod'а с одним identity = опасно)

#### Ручное вмешательство

1. Убедиться что нода действительно **failed**
2. `kubectl delete pod quiz-1 --force --grace-period 0`
3. Новый pod создаётся **controller'ом**
4. Если **PV local** → pod не может быть scheduled на другую ноду
   - Удалить **PVC + pod** → новый PVC + новый PV

^sts-node-failure

## Pod Management Policy

#### OrderedReady (default)

- **Scale up:** quiz-0 → ready → quiz-1 → ready → quiz-2
- **Scale down:** quiz-2 → terminated → quiz-1 → terminated → quiz-0
- **minReadySeconds:** задержка между pod'ами

> Если pod-0 **не ready** → pod-1 **никогда не создастся** (deadlock!)
> **Template update** НЕ применяется к **unready** pod'ам
> **Scale down** блокируется если не все pod'ы ready
> НЕ применяется при **удалении** StatefulSet

#### Parallel

- **Scale up:** quiz-0, quiz-1, quiz-2 — **все одновременно**
- **Scale down:** все удаляются одновременно
- Быстрее, но **не все приложения поддерживают**

> **podManagementPolicy** — **immutable**. Чтобы изменить: `delete sts --cascade=orphan` → recreate

^sts-pod-management

## Полезные команды

```bash
kubectl get sts                        # список StatefulSets
kubectl get sts -o wide                # + containers, images
kubectl describe sts quiz
kubectl rollout status sts quiz
kubectl scale sts quiz --replicas 5
kubectl get pvc -l app=quiz            # PVC StatefulSet'а
kubectl delete pod quiz-1 --force --grace-period 0  # при node failure
```

^sts-commands

## Связь
- [[StatefulSet — updates и Operators]] — update strategies, partition, Operators (Ch.16)
- [[Service — DNS, endpoints, readiness]] — headless Service для peer discovery (Ch.11)
- [[PV, PVC, StorageClass]] — volumeClaimTemplates создают PVC (Ch.10)
- [[ReplicaSet]] — StatefulSet vs ReplicaSet: ordinal vs random names (Ch.14)
- [[Deployment — стратегии обновления]] — Deployment для stateless, StatefulSet для stateful (Ch.15)
