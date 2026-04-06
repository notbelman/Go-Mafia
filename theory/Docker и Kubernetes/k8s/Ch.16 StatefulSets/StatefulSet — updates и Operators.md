- **Update strategies:** RollingUpdate (default — по одному, highest ordinal first, нет maxSurge/maxUnavailable) и OnDelete (semi-auto — ты удаляешь pod, controller заменяет)
- **partition** — аналог pause для StatefulSet. Pod'ы с ordinal < partition НЕ обновляются. Canary: partition = replicas-1 (обновится только последний pod). Staged rollout: partition = replicas (никто не обновляется), затем снижаешь постепенно
- **Revision history** хранится в **ControllerRevision** объектах (не в ReplicaSet'ах как у Deployment). Rollback через `kubectl rollout undo`
- **Kubernetes Operator** = application-specific controller. Расширяет K8s API через CustomResourceDefinition. Автоматизирует lifecycle (scaling, backup, failover)
- StatefulSet ≠ полное управление stateful app. Operator закрывает gap: например для MongoDB: инициализация replica set, добавление членов при scaling, автоматический failover

---

## Update Strategies

| Параметр | **RollingUpdate** (default) | **OnDelete** |
|---|---|---|
| **Автоматизация** | автоматически | semi-auto (ты удаляешь pod) |
| **Порядок** | highest ordinal → lowest | любой (ты выбираешь) |
| **maxSurge/maxUnav.** | НЕТ (всегда 1 за раз) | N/A |
| **minReadySeconds** | да (задержка после ready) | не применяется |
| **Partition** | да (canary, staging) | N/A |
| **Faulty version** | блокирует rollout | ты контролируешь |

^sts-update-strategies

## RollingUpdate

#### Порядок обновления

**Reverse ordinal**: quiz-2 → quiz-1 → quiz-0

```
Timeline:
  quiz-0 (v1)  quiz-1 (v1)  quiz-2 (v1)
  quiz-0 (v1)  quiz-1 (v1)  quiz-2 (v2) ← highest first
  quiz-0 (v1)  quiz-1 (v2)  quiz-2 (v2)
  quiz-0 (v2)  quiz-1 (v2)  quiz-2 (v2) ← complete
```

#### Отличия от Deployment RollingUpdate

- Только **1 pod за раз** (нет maxSurge/maxUnavailable)
- Нет нового **ReplicaSet** (pod'ы обновляются **in-place**)
- **partition** вместо pause
- Если pod **not ready** → rollout **блокируется**
- Если pod **not ready ДО update** → rollout тоже **блокируется**

^sts-rolling

## Partition (Canary / Staging)

```yaml
spec:
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      partition: 2             # pod'ы с ordinal < 2 НЕ обновляются
```

^sts-partition-manifest

**partition = N** → обновляются только pod'ы с **ordinal >= N**

#### Stage update (не запускать rollout)

- **partition: 3** (= replicas) → ни один pod не обновляется
- Вносим изменения в **template**
- Когда готовы → уменьшаем **partition**

#### Canary

- **partition: 2** (= replicas - 1) → только **quiz-2** обновляется
- Проверяем **canary pod**
- **partition: 0** → все pod'ы обновляются

#### Phased rollout

- **partition: 2** → quiz-2 updated
- **partition: 1** → quiz-1 updated
- **partition: 0** → quiz-0 updated

> [!warning]
> Если удалить pod с **ordinal < partition** → он пересоздаётся со **СТАРЫМ** template
> Если удалить pod с **ordinal >= partition** → пересоздаётся с **НОВЫМ** template

^sts-partition

## OnDelete Strategy

```yaml
spec:
  updateStrategy:
    type: OnDelete              # нет параметров
```

^sts-ondelete-manifest

#### Workflow

1. Обновляем **template** в StatefulSet → ничего не происходит
2. `kubectl delete pod quiz-0` → controller создаёт quiz-0 с **НОВЫМ template**
3. Повторяем для остальных pod'ов в **любом порядке**

#### Rollback

**`kubectl rollout undo`** — тоже **semi-auto**: template откатывается, но pod'ы нужно удалять **вручную**

#### Use case

- **Полный контроль** над порядком и временем обновления
- Можно обновлять в **произвольном порядке**
- **Readiness status** не важен (controller заменяет любой удалённый pod)

^sts-ondelete

## Revision History

- **Deployment**: история в **ReplicaSet'ах** (replicas: 0)
- **StatefulSet**: история в **ControllerRevision** объектах

```bash
kubectl rollout history sts quiz
kubectl rollout history sts quiz --revision 2
kubectl get controllerrevisions

kubectl rollout undo sts quiz                    # предыдущая ревизия
kubectl rollout undo sts quiz --to-revision=1    # конкретная ревизия
```

> [!warning]
> Rollback соблюдает текущую **update strategy**:
> - **RollingUpdate** → постепенный rollback
> - **OnDelete** → semi-auto rollback (ты удаляешь pod'ы)

^sts-revision

## Kubernetes Operators

#### Проблема

**StatefulSet** не покрывает ВСЁ для **stateful apps**:
- **Инициализация replica set** (MongoDB `rs.initiate`)
- **Reconfiguration** при scaling (добавить/удалить members)
- **Automated failover** при node failure
- **Backup / restore**
- **Version upgrades** с data migration

#### Operator = application-specific controller

1. Расширяет K8s API через **CustomResourceDefinition (CRD)**
2. Создаёт **custom object kind** (например: `MongoDBCommunity`)
3. **Controller** (operator) следит за custom objects
4. Создаёт/управляет **StatefulSet**, **Services**, **Secrets** автоматически

```
User ──▶ MongoDBCommunity object ──▶ Operator ──▶ StatefulSet + Services
         (declarative spec)                       (managed automatically)
```

^sts-operator-concept

#### Пример: MongoDB Community Operator

```yaml
# Custom Resource:
apiVersion: mongodbcommunity.mongodb.com/v1
kind: MongoDBCommunity
metadata:
  name: my-mongodb
spec:
  members: 3                     # количество реплик
  type: ReplicaSet
  version: "5.0"
```

**Operator автоматически:**
- Создаёт **StatefulSet** с 3 pod'ами
- Создаёт **headless Service**
- Инициирует **MongoDB replica set**
- При scale (изменить **members**) → **reconfigure** replica set
- **Managed failover**

#### Другие популярные Operators

| **Database** | **Operator** |
|---|---|
| **PostgreSQL** | Zalando Postgres Operator, CrunchyData PGO |
| **Redis** | Redis Operator |
| **Elasticsearch** | ECK (Elastic Cloud on Kubernetes) |
| **Kafka** | Strimzi |
| **MySQL** | Oracle MySQL Operator |

^sts-operator-example

## Связь
- [[StatefulSet — концепт]] — что такое StatefulSet, headless Service, PVC templates (Ch.16)
- [[Deployment — стратегии обновления]] — RollingUpdate в Deployment vs StatefulSet (Ch.15)
- [[Deployment — rollback и стратегии]] — rollback, revision history (Ch.15)
- [[Kubernetes API и манифесты]] — CRD расширяют API (Ch.4)
