- **Deployment** = ReplicaSet + автоматическое обновление pod'ов. Обновил pod template → Deployment controller создаёт новый ReplicaSet и переводит pod'ы
- Две встроенные стратегии: **Recreate** (все pod'ы удаляются, потом создаются — downtime) и **RollingUpdate** (постепенная замена — без downtime, default)
- **maxSurge** = сколько pod'ов СВЕРХ desired можно создать. **maxUnavailable** = сколько pod'ов НИЖЕ desired допустимо. Default: оба 25%
- **pod-template-hash** — label, добавляемый Deployment controller'ом. Значение = hash от pod template. Каждый новый template → новый ReplicaSet с новым hash
- **minReadySeconds** — pod должен быть ready N секунд перед тем как считаться available. Защита от faulty versions: pod не available → rollout не продолжается

---

## Deployment vs ReplicaSet

```
ReplicaSet:                         Deployment:
  Управляет pod'ами                   Управляет ReplicaSet'ами
  Template update → ничего            Template update → rolling update
  Нет rollback                        Rollback через revision history
  Используй напрямую? → НЕТ          Используй для stateless workloads

Deployment → ReplicaSet → Pods
  (ты)        (auto)      (auto)
```
^dep-vs-rs

## Manifest

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kiada
spec:
  replicas: 3
  strategy:
    type: RollingUpdate            # или Recreate
    rollingUpdate:
      maxSurge: 1                  # max pod'ов сверх desired
      maxUnavailable: 0            # max недоступных pod'ов
  minReadySeconds: 10              # секунд ready перед available
  selector:
    matchLabels:
      app: kiada
      rel: stable
  template:
    metadata:
      labels:
        app: kiada
        rel: stable
        ver: "0.5"
    spec:
      containers:
      - name: kiada
        image: luksa/kiada:0.5
```
^dep-manifest

## Recreate Strategy

```
spec:
  strategy:
    type: Recreate

Timeline:
  v1 v1 v1 ──── все удалены ──── v2 v2 v2
                 ↑ DOWNTIME ↑

1. Deployment controller масштабирует старый RS до 0
2. Все старые pod'ы удаляются одновременно
3. Создаётся новый RS с desired replicas
4. Новые pod'ы стартуют одновременно

Use case: приложение НЕ может работать в двух версиях одновременно
Минус: downtime (503 Service Temporarily Unavailable)
```
^dep-recreate

## RollingUpdate Strategy

```
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 0
      maxUnavailable: 1

Timeline (maxSurge=0, maxUnavailable=1):
  v1 v1 v1    → v1 v1 __   → v1 v1 v2   → v1 __ v2
              → v1 v2 v2   → __ v2 v2   → v2 v2 v2

  Одновременно максимум 3 pod'а (maxSurge=0)
  Минимум 2 available (maxUnavailable=1)

Каждый шаг:
  1. Scale down old RS by 1
  2. Scale up new RS by 1
  3. Ждать пока новый pod ready + minReadySeconds
  4. Повторить
```
^dep-rolling

## maxSurge и maxUnavailable

```
desired = 3

maxSurge=0, maxUnavailable=1:
  total ≤ 3, available ≥ 2
  → медленно, по одному pod'у за раз

maxSurge=1, maxUnavailable=0:
  total ≤ 4, available ≥ 3
  → сначала создать новый, потом удалить старый
  → zero downtime guarantee

maxSurge=1, maxUnavailable=1:
  total ≤ 4, available ≥ 2
  → быстрее (2 pod'а за раз)

maxSurge=0, maxUnavailable=0:
  ❌ НЕВАЛИДНО (невозможно обновить)

Default: maxSurge=25%, maxUnavailable=25%
  При 10 replicas: maxSurge=3 (ceil), maxUnavailable=2 (floor)
  total ≤ 13, available ≥ 8
```
^dep-surge-unavailable

## Как Deployment создаёт ReplicaSets

```
Deployment: kiada
  │
  ├── RS: kiada-7bffb9bf96 (hash от template v0.5)
  │     replicas: 0 (старый, после update)
  │     pod-template-hash: 7bffb9bf96
  │
  └── RS: kiada-58df67c6f6 (hash от template v0.6)
        replicas: 3 (текущий, active)
        pod-template-hash: 58df67c6f6

Pod names: kiada-58df67c6f6-4knb6
           ─────┬───────── ──┬──
           RS name           random suffix

pod-template-hash label:
  → добавляется к RS и pod'ам автоматически
  → значение = hash от pod template
  → обеспечивает уникальность RS для каждой версии template
  → предотвращает "захват" pod'ов старым RS
```
^dep-replicasets

## minReadySeconds — защита от faulty versions

```
minReadySeconds: 60
  → pod ready → ждём 60 секунд → pod available → rollout продолжается
  → если pod fails readiness probe за эти 60s → таймер сбрасывается
  → rollout НЕ продолжается пока pod не станет available

Пример:
  Faulty version: приложение падает через 30s
  minReadySeconds: 60
  → pod стартует, ready, но через 30s fails readiness probe
  → pod никогда не available → rollout застревает
  → старые pod'ы продолжают обслуживать трафик
  → ты можешь rollback

Без minReadySeconds:
  → pod ready → сразу available → rollout продолжается
  → все pod'ы обновлены → все падают через 30s → полный outage
```
^dep-minready

## Связь
- [[Deployment — rollback и стратегии]] — rollback, pause, canary, blue/green (Ch.15)
- [[ReplicaSet]] — RS как building block Deployment'а (Ch.14)
- [[Probes и Lifecycle Hooks]] — readiness probe определяет ready → available (Ch.6)
- [[Service — типы и routing]] — Service раздаёт трафик pod'ам обоих версий при rolling update (Ch.11)
