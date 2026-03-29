- **StatefulSet** = Deployment для stateful workloads. Pod'ы получают **ordinal имена** (quiz-0, quiz-1...), каждый — свой **PersistentVolumeClaim**. Pets vs Cattle: Deployment = cattle (взаимозаменяемы), StatefulSet = pets (уникальная identity)
- **Headless Service** (clusterIP: None) + StatefulSet → каждый pod получает DNS record: `quiz-0.quiz-pods.ns.svc.cluster.local`. Stable network identity при пересоздании pod'а
- **volumeClaimTemplates** — PVC создаётся для каждого pod'а автоматически. Scale down → PVC сохраняются (default). Scale up → pod'ы переподключаются к тем же PVC
- **At-most-one semantics:** StatefulSet НЕ создаёт replacement pod при node failure (в отличие от ReplicaSet). Нужно ручное удаление `--force --grace-period 0`
- **podManagementPolicy:** OrderedReady (default — по одному, ждёт ready) vs Parallel (все сразу). OrderedReady может привести к deadlock

---

## StatefulSet vs Deployment

```
                        Deployment              StatefulSet
Pod names               random (hash+suffix)    ordinal (quiz-0, quiz-1...)
Pod identity            fungible (cattle)       stable (pets)
PVC per pod             нет (общий PVC)         да (volumeClaimTemplates)
Pod creation order      все сразу               по одному (OrderedReady) или сразу
Scale down order        по правилам             highest ordinal first
DNS per pod             нет                     да (через headless Service)
Node failure            auto replacement        НЕТ auto replacement (at-most-one)
Update strategy         Recreate/RollingUpdate  RollingUpdate/OnDelete
Owns pods via           ReplicaSet              напрямую (нет RS)
```
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

```
DNS records:
  quiz-pods.kiada.svc.cluster.local          → IPs всех pod'ов
  quiz-0.quiz-pods.kiada.svc.cluster.local   → IP quiz-0
  quiz-1.quiz-pods.kiada.svc.cluster.local   → IP quiz-1
  quiz-2.quiz-pods.kiada.svc.cluster.local   → IP quiz-2

SRV records (peer discovery):
  _mongodb._tcp.quiz-pods.kiada.svc.cluster.local
  → используется MongoDB client: mongodb+srv://quiz-pods.kiada.svc.cluster.local

Два Service'а — типичный паттерн:
  quiz-pods (headless) — peer discovery, publishNotReadyAddresses: true
  quiz (regular)       — client traffic, только ready pod'ы
```
^sts-dns

## Pod Names и PVC Names

```
StatefulSet: quiz, replicas: 3

Pods:   quiz-0    quiz-1    quiz-2
PVCs:   db-data-quiz-0  db-data-quiz-1  db-data-quiz-2
        ───────┬──────   ──────┬──────
        template name    pod name

Pod удалён → новый pod с ТЕМ ЖЕ именем → ТОТ ЖЕ PVC
  → state сохраняется при пересоздании

Labels добавляемые controller'ом:
  controller-revision-hash: quiz-7576f64fbc    (как pod-template-hash)
  statefulset.kubernetes.io/pod-name: quiz-0   (для per-pod Service)

ownerReferences → StatefulSet напрямую (не через ReplicaSet)
```
^sts-naming

## Scaling

```
Scale up:
  → новые pod'ы + новые PVC создаются
  → ordinal numbers продолжаются (quiz-3, quiz-4...)

Scale down:
  → pod с НАИБОЛЬШИМ ordinal удаляется первым
  → PVC по умолчанию СОХРАНЯЮТСЯ (Retain)
  → scale up обратно → pod переподключается к тому же PVC

Scale down to 0:
  → все pod'ы удалены, PVC остаются
  → scale up → все PVC переподключаются

⚠️ Stateful приложения могут требовать доп. конфигурации при scaling
   (например, MongoDB replica set reconfiguration)
```
^sts-scaling

## PVC Retention Policy

```yaml
spec:
  persistentVolumeClaimRetentionPolicy:
    whenScaled: Retain      # default: Retain. При scale down
    whenDeleted: Retain      # default: Retain. При удалении StatefulSet

# whenScaled: Delete → PVC удаляются при scale down
# whenDeleted: Delete → PVC удаляются при удалении StatefulSet

⚠️ whenScaled: Delete + scale to 0 → все данные потеряны
⚠️ Лучше: Retain + ручное удаление PVC
```
^sts-pvc-retention

## Node Failure — At-Most-One Semantics

```
ReplicaSet при node failure:
  → через несколько минут создаёт replacement pod на другой ноде
  → возможно ДВА экземпляра с одинаковыми данными работают одновременно

StatefulSet при node failure:
  → pod отмечается как Terminating
  → НЕ создаёт replacement автоматически
  → причина: at-most-one guarantee (два pod'а с одним identity = опасно)

Ручное вмешательство:
  1. Убедиться что нода действительно failed
  2. kubectl delete pod quiz-1 --force --grace-period 0
  3. Новый pod создаётся controller'ом
  4. Если PV local → pod не может быть scheduled на другую ноду
     → удалить PVC + pod → новый PVC + новый PV
```
^sts-node-failure

## Pod Management Policy

```
OrderedReady (default):
  Scale up:   quiz-0 → ready → quiz-1 → ready → quiz-2
  Scale down: quiz-2 → terminated → quiz-1 → terminated → quiz-0
  minReadySeconds: задержка между pod'ами

  ⚠️ Если pod-0 не ready → pod-1 НИКОГДА не создастся (deadlock!)
  ⚠️ Template update НЕ применяется к unready pod'ам
  ⚠️ Scale down блокируется если не все pod'ы ready
  ⚠️ НЕ применяется при удалении StatefulSet

Parallel:
  Scale up:   quiz-0, quiz-1, quiz-2 — все одновременно
  Scale down: все удаляются одновременно
  → быстрее, но не все приложения поддерживают

⚠️ podManagementPolicy — immutable
   → чтобы изменить: delete sts --cascade=orphan → recreate
```
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
