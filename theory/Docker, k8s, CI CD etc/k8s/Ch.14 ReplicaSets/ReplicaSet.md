- **ReplicaSet** = гарантирует что N pod'ов с нужными labels всегда запущены. Spec: replicas + selector + pod template. Основа Deployment'ов
- **Reconciliation loop:** controller наблюдает за pod'ами → actual ≠ desired → создаёт/удаляет pod'ы. Работает постоянно, реагирует на любые изменения
- Pod'ы ReplicaSet'а **fungible** (взаимозаменяемы) — random names, нет порядка. Для stateful → StatefulSet
- **ownerReferences** — pod'ы принадлежат ReplicaSet'у. Удаление ReplicaSet → garbage collector удаляет pod'ы. `--cascade=orphan` сохраняет pod'ы
- Обновление pod template **НЕ обновляет** существующие pod'ы — только новые pod'ы создаются по новому template. Для rolling updates → Deployment

---

## ReplicaSet Manifest

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: kiada
spec:
  replicas: 3                    # желаемое количество pod'ов
  selector:                      # какие pod'ы принадлежат этому RS
    matchLabels:
      app: kiada
      rel: stable
  template:                      # шаблон для создания pod'ов
    metadata:
      labels:
        app: kiada               # ДОЛЖНЫ совпадать с selector
        rel: stable
        ver: "0.5"               # могут быть доп. labels
    spec:
      containers:
      - name: kiada
        image: luksa/kiada:0.5
        ...
```
^rs-manifest

**Имена pod'ов:** `kiada-` + random suffix (generateName). Без порядковых номеров — pod'ы взаимозаменяемы. ^rs-naming

## Reconciliation Loop

```
                    ┌──────────────────────────┐
                    │    ReplicaSet Controller  │
                    └──────────┬───────────────┘
                               │
              ┌────────────────▼────────────────┐
              │  Watch: ReplicaSet + Pod objects │
              └────────────────┬────────────────┘
                               │
              ┌────────────────▼────────────────┐
              │  Count pods matching selector   │
              └────────────────┬────────────────┘
                               │
                ┌──────────────┼──────────────┐
                ▼              ▼              ▼
          actual < desired  actual == desired  actual > desired
                │              │              │
          Create pods     Do nothing     Delete excess pods
          from template                  (по правилам приоритета)
```
^rs-reconciliation

## Scaling

```bash
kubectl scale rs kiada --replicas 5     # scale up
kubectl scale rs kiada --replicas 0     # scale down to zero (pod'ы удалены, RS остаётся)
kubectl edit rs kiada                   # изменить replicas вручную
```
^rs-scaling

### Порядок удаления при scale down

```
1. Pod'ы без назначенной ноды
2. Pod'ы с phase Unknown
3. Pod'ы которые не ready
4. Pod'ы с меньшим deletion cost (annotation controller.kubernetes.io/pod-deletion-cost)
5. Pod'ы на нодах с большим количеством реплик (выравнивание по нодам)
6. Pod'ы которые были ready меньше времени
7. Pod'ы с большим количеством restarts
8. Pod'ы созданные позже
```
^rs-deletion-order

## Pod Ownership

```yaml
# Pod's metadata:
ownerReferences:
- apiVersion: apps/v1
  kind: ReplicaSet
  name: kiada
  controller: true          # этот owner управляет pod'ом
  blockOwnerDeletion: true

# Удаление RS → garbage collector удаляет pod'ы
# kubectl delete rs kiada --cascade=orphan → pod'ы сохраняются
```
^rs-ownership

## Обновление Template

```
Изменил pod template в RS → существующие pod'ы НЕ обновляются
  → только НОВЫЕ pod'ы (при scale up или замене) используют новый template
  → RS "cookie cutter" — меняется форма, но уже вырезанное не меняется

Для автоматического обновления → используй Deployment
```
^rs-template-update

## Удаление из RS без удаления pod'а

```bash
# Изменить label pod'а → он выпадает из RS → RS создаёт замену
kubectl label pod kiada-78j7m rel=debug --overwrite

# Полезно: pod сломался → убрать из RS для debug
# RS создаст новый pod, а сломанный можно исследовать
```
^rs-remove-pod

## RS НЕ гарантирует healthy pod'ы

```
RS гарантирует: actual count == desired count
RS НЕ гарантирует: все pod'ы ready/healthy

Если pod crash-loop'ит или fail'ит readiness probe:
  → RS НЕ удаляет и НЕ заменяет его
  → RS считает: pod существует → count совпадает → всё ок
  → ты должен сам удалить/починить проблемный pod
```
^rs-not-healthy

## Полезные команды

```bash
kubectl get rs                         # список ReplicaSets
kubectl get rs -o wide                 # + containers, images, selector
kubectl describe rs kiada              # детали + events
kubectl get pods -l app=kiada          # pod'ы по selector
kubectl logs rs/kiada -c kiada         # логи одного pod'а через RS
kubectl logs rs/kiada --all-pods -c kiada -f  # логи всех pod'ов
```
^rs-commands

## Связь
- [[Deployment — стратегии обновления]] — Deployment управляет ReplicaSet'ами (Ch.15)
- [[Namespaces, labels, selectors, annotations]] — selector связывает RS с pod'ами (Ch.7)
- [[Service — типы и routing]] — Service тоже выбирает pod'ы по labels (Ch.11)
- [[Pod — что это и зачем]] — pod template определяет pod'ы (Ch.5)
- [[StatefulSet — концепт]] — RS для stateless, StatefulSet для stateful (Ch.16)
