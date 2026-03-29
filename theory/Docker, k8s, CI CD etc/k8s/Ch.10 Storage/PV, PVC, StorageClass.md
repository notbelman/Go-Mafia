- **PersistentVolume (PV)** — объект K8s, представляющий storage volume (local или network). **PersistentVolumeClaim (PVC)** — "заявка" пользователя на storage. Pod → PVC → PV → underlying storage
- **StorageClass** — определяет КАК провизионить volumes (provisioner + parameters). Кластер может иметь несколько классов (standard, premium, local). Один — default
- **Dynamic provisioning** (default): создал PVC → provisioner автоматически создал PV + underlying storage. **Static**: admin заранее создал PV, K8s матчит с PVC
- **Access modes:** RWOP (один pod), RWO (одна нода, много pod'ов), RWX (много нод), ROX (много нод, read-only)
- **Reclaim policy:** Delete (dynamic default — PV и storage удаляются с PVC), Retain (static default — PV остаётся, admin чистит вручную)

---

## Связь объектов

```
Pod                PVC               PV              Storage
┌──────────┐      ┌──────────┐     ┌──────────┐    ┌──────────┐
│ volumes:  │      │ spec:    │     │ spec:    │    │          │
│  - pvc:   │─────▶│  storage │────▶│  capacity│───▶│ NFS/EBS/ │
│   name: X │ ref  │  access  │bind │  access  │    │ GCE PD / │
│           │      │  class   │     │  local   │    │ local    │
└──────────┘      └──────────┘     └──────────┘    └──────────┘
  namespaced        namespaced      cluster-scoped

Pod ссылается на PVC по имени
PVC ссылается на StorageClass по имени
PV создаётся автоматически (dynamic) или заранее (static)
```
^pv-relationship

## PersistentVolumeClaim (PVC)

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-data
spec:
  storageClassName: standard-rwo   # какой класс storage (default если пусто)
  resources:
    requests:
      storage: 10Gi               # минимальный размер
  accessModes:
  - ReadWriteOncePod               # режим доступа
  volumeMode: Filesystem           # Filesystem (default) или Block
  # dataSourceRef:                 # клонирование из другого PVC или snapshot
  #   kind: PersistentVolumeClaim
  #   name: source-pvc
```
^pv-pvc-manifest

## Использование PVC в Pod'е

```yaml
spec:
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: my-data           # имя PVC
      readOnly: false              # опционально
  containers:
  - name: app
    volumeMounts:
    - name: data
      mountPath: /data/db
```
^pv-pod-usage

## Access Modes

```
Mode              Abbr    Кто может монтировать
──────────────────────────────────────────────────────
ReadWriteOncePod  RWOP    Один pod (R/W) во всём кластере
ReadWriteOnce     RWO     Одна нода (R/W), но много pod'ов на ней
ReadWriteMany     RWX     Много нод (R/W) — не все storage поддерживают
ReadOnlyMany      ROX     Много нод (Read-only)

⚠️ RWO ≠ RWOP:
  RWO → один NODE, но несколько pod'ов на нём могут писать
  RWOP → один POD во всём кластере

⚠️ ReadOnlyOnce не существует
  → используй RWO volume с readOnly: true в pod manifest
```
^pv-access-modes

## Dynamic vs Static Provisioning

```
Dynamic (большинство кластеров):
  1. User создаёт PVC (указывает StorageClass, размер, access mode)
  2. CSI provisioner автоматически создаёт PV + underlying storage
  3. PVC привязывается (Bound) к PV
  4. Pod использует PVC
  5. Удаление PVC → PV и storage удаляются (reclaim policy: Delete)

Static:
  1. Admin создаёт underlying storage (NFS share, local disk)
  2. Admin создаёт PV объект (указывает на storage)
  3. User создаёт PVC с требованиями
  4. K8s находит подходящий PV и привязывает
  5. Удаление PVC → PV переходит в Released (reclaim policy: Retain)
     → admin должен вручную очистить и пересоздать PV
```
^pv-provisioning

## StorageClass

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard-rwo
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"  # default class
provisioner: pd.csi.storage.gke.io     # CSI driver
parameters:
  type: pd-balanced                     # параметры для provisioner
reclaimPolicy: Delete                   # Delete или Retain
volumeBindingMode: WaitForFirstConsumer # когда создавать PV
allowVolumeExpansion: true              # можно ли увеличивать size
```
^pv-storageclass

```
kubectl get sc                    # список StorageClasses
                                  # (default) — класс по умолчанию

volumeBindingMode:
  Immediate           → PV создаётся сразу при создании PVC
  WaitForFirstConsumer → PV создаётся когда первый Pod использует PVC
                        (нужен для local volumes и topology-aware storage)
```
^pv-binding-mode

## Reclaim Policy

```
Delete (default для dynamic):
  PVC удалён → PV удалён → underlying storage удалён
  → данные ПОТЕРЯНЫ

Retain (default для static):
  PVC удалён → PV переходит в Released
  → underlying storage НЕ удалён
  → PV нельзя привязать к новому PVC (статус Released, не Available)
  → чтобы переиспользовать: удалить PV и создать заново
    (или убрать claimRef из spec)

⚠️ Если PV в Released и меняешь policy с Retain на Delete → PV и storage удалятся
💡 Перед удалением PVC с policy Delete → смени на Retain чтобы сохранить данные
```
^pv-reclaim-policy

## Lifecycle PV

```
PV statuses:
  Available → свободен, можно привязать
  Bound     → привязан к PVC
  Released  → PVC удалён, но PV ещё существует (Retain policy)
  Failed    → ошибка автоматической очистки

PVC statuses:
  Pending   → ожидает привязки (нет подходящего PV или WaitForFirstConsumer)
  Bound     → привязан к PV
  Terminating → удаляется (ждёт пока pod'ы отпустят)

Удаление PV/PVC пока pod использует → блокируется (Terminating)
K8s НИКОГДА не убивает pod'ы из-за удаления PV/PVC
```
^pv-lifecycle

## Local PersistentVolumes

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-disk
spec:
  storageClassName: local
  capacity:
    storage: 10Gi
  accessModes: [ReadWriteOnce]
  local:
    path: /mnt/my-disk              # путь на ноде
  nodeAffinity:                     # на какой ноде диск
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values: [worker-1]
```
^pv-local

```
Local PV vs hostPath volume:
  Local PV:    scheduler привязывает pod к ноде с диском, admin контролирует
  hostPath:    pod может попасть на любую ноду, доступ к произвольным путям
  → Local PV безопаснее и предсказуемее
```
^pv-local-vs-hostpath

## Resize, Snapshots, Cloning

```
Resize:
  → увеличить spec.resources.requests.storage в PVC
  → StorageClass должен иметь allowVolumeExpansion: true
  → может потребовать restart pod'а (FileSystemResizePending condition)
  → уменьшить нельзя

Cloning (dataSourceRef):
  → новый PVC с dataSourceRef → kind: PersistentVolumeClaim
  → создаёт копию данных из source PVC

Snapshots:
  → VolumeSnapshotClass → VolumeSnapshot → VolumeSnapshotContent
  → аналогия: StorageClass → PVC → PV
  → восстановление: новый PVC с dataSourceRef → kind: VolumeSnapshot

Ephemeral volumes:
  → volumeClaimTemplate в pod spec
  → PVC создаётся и удаляется ВМЕСТЕ с pod'ом
  → как emptyDir, но с фиксированным размером и фичами PV
```
^pv-management

## CSI Drivers

```
Container Storage Interface — стандартный интерфейс для storage plugins
  → код storage вне ядра K8s ("out-of-tree")
  → каждый driver = controller (provisioning) + node agent (mount/unmount)

kubectl get csidrivers        # установленные CSI drivers
kubectl get sc                # StorageClasses ссылаются на CSI drivers

Примеры:
  pd.csi.storage.gke.io       → Google Persistent Disk
  ebs.csi.aws.com             → AWS EBS
  disk.csi.azure.com          → Azure Disk
  nfs.csi.k8s.io              → NFS
```
^pv-csi

## Полезные команды

```bash
kubectl get pvc                    # список PersistentVolumeClaims
kubectl get pv                     # список PersistentVolumes
kubectl get sc                     # список StorageClasses
kubectl describe pvc my-data       # детали + conditions + events
kubectl get pvc -o wide            # расширенный вывод
```
^pv-commands

## Связь
- [[ConfigMaps]] — configMap volume для конфиг-файлов (Ch.8)
- [[Secrets и Downward API]] — secret volume для чувствительных файлов (Ch.8)
- [[StatefulSet — концепт]] — volumeClaimTemplates для stateful приложений (Ch.16)
- [[Pod — что это и зачем]] — volumes в pod spec (Ch.5)
- [[Архитектура кластера]] — CSI driver как компонент кластера (Ch.1)
