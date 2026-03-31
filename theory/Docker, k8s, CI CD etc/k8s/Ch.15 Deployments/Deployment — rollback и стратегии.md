- **Rollback:** `kubectl rollout undo` — откатывает pod template к предыдущей ревизии (или конкретной через `--to-revision`). Стратегия обновления соблюдается при откате
- **Revision history** хранится в старых ReplicaSet'ах. `revisionHistoryLimit` (default 10) определяет сколько RS хранить. `kubectl rollout history` показывает ревизии
- **Pause/Resume:** `kubectl rollout pause` — останавливает rollout (можно проверить canary pod'ы). `kubectl rollout resume` — продолжает. Pause перед update → batch изменения
- **Canary:** два Deployment'а (stable + canary) с разным replicas count, один Service с общим selector. Или single Deployment + minReadySeconds + pause: rollout обновит 1 pod и остановится, можно проверить canary перед продолжением
- **Blue/Green:** два Deployment'а (blue + green), Service selector переключается между ними. Мгновенный switch трафика

---

## Rollback

```bash
# Откатить к предыдущей ревизии:
kubectl rollout undo deployment kiada

# Откатить к конкретной ревизии:
kubectl rollout undo deployment kiada --to-revision=2
```

- **Rollback** = обратный update (соблюдает **strategy**)
  - **RollingUpdate** → постепенный откат
  - **Recreate** → все pod'ы заменяются сразу

> ⚠️ `undo` откатывает **ТОЛЬКО pod template**. **replicas**, **strategy**, **minReadySeconds** — **НЕ** откатываются. Используй `kubectl apply` если нужно откатить **ВСЁ** включая strategy/replicas.

^dep-rollback

## Revision History

```bash
# Посмотреть историю ревизий:
kubectl rollout history deploy kiada

# Посмотреть конкретную ревизию:
kubectl rollout history deploy kiada --revision 2

# Лучший способ — list RS с деталями:
kubectl get rs -o wide -L ver
```

- **Revision number** хранится в annotation: `deployment.kubernetes.io/revision`
- **История** хранится в старых **ReplicaSet'ах** (replicas: 0)
- **revisionHistoryLimit: 10** (default) → максимум **10 RS** хранятся

^dep-history

```
Deployment
  │
  ├── RS rev.1 (replicas: 0, image: v0.5)  ← история
  ├── RS rev.2 (replicas: 0, image: v0.6)  ← история
  └── RS rev.3 (replicas: 3, image: v0.7)  ← текущий (active)

Rollback to rev.1:
  RS rev.1 → scale up to 3
  RS rev.3 → scale down to 0
  rev.1 получает новый номер (rev.4)
```
^dep-history-diagram

## Pause / Resume

```bash
# Pause rollout (в процессе обновления):
kubectl rollout pause deployment kiada

# Resume:
kubectl rollout resume deployment kiada
```

- **Pause в процессе обновления** → rollout останавливается, часть pod'ов **old**, часть **new** → можно проверить **new pod'ы**
- **Pause перед обновлением** → вносим несколько изменений (**image**, **env**, **labels**...) → `kubectl rollout resume` → все изменения применяются в **одном rollout**

^dep-pause

## Отслеживание Rollout

```bash
# Статус rollout:
kubectl rollout status deploy kiada

# Детали и conditions:
kubectl describe deploy kiada

# Restart pod'ов (без изменения template):
kubectl rollout restart deployment kiada
```

#### Conditions

| Condition | Status | Reason | Значение |
|-----------|--------|--------|----------|
| **Progressing** | True | NewReplicaSetAvailable | Rollout **завершён** |
| **Available** | True | MinimumReplicasAvailable | Достаточно **ready** pod'ов |
| **Progressing** | False | ProgressDeadlineExceeded | Rollout **застрял** > **progressDeadlineSeconds** (default **600s**) |

- `kubectl rollout restart` → pod'ы **пересоздаются** по текущей **strategy**

^dep-rollout-status

## Deployment Strategies — расширенные

#### Встроенные в K8s

| Стратегия | Описание |
|-----------|----------|
| **Recreate** | Все pod'ы удаляются, потом создаются (**downtime**) |
| **RollingUpdate** | Постепенная замена (**default**, **zero downtime**) |

#### Реализуемые вручную через K8s примитивы

| Стратегия | Описание |
|-----------|----------|
| **Canary** | Два Deployment'а + один **Service** |
| **Blue/Green** | Два Deployment'а + **Service selector switch** |
| **A/B Testing** | Два Deployment'а + **Ingress routing** |

#### Требуют внешних инструментов

| Стратегия | Описание |
|-----------|----------|
| **Traffic Shadowing** | Ingress / **Service Mesh** mirroring |
| **Advanced Canary** | **Flagger**, **Argo Rollouts** |

^dep-strategies-overview

## Canary

```
kiada-stable Deployment (replicas: 9)     kiada-canary Deployment (replicas: 1)
  selector: app=kiada, rel=stable           selector: app=kiada, rel=canary

        kiada Service
        selector: app=kiada       ← матчит оба Deployment'а
        → 90% трафика → stable pod'ы (9 штук)
        → 10% трафика → canary pod (1 штука)

Проверили canary → обновляем stable Deployment
→ удаляем canary Deployment
```
^dep-canary

## Blue/Green

```
Step 1: Blue active
  kiada-blue Deployment:   app=kiada, col=blue   (v1)
  kiada-green Deployment:  app=kiada, col=green  (v2, готовится)
  Service selector:        app=kiada, col=blue    ← трафик на blue

Step 2: Switch traffic
  kubectl set selector svc kiada app=kiada,col=green
  → мгновенный switch ВСЕГО трафика на green (v2)

Step 3: Cleanup
  → если всё ок → удалить blue Deployment
  → если проблемы → switch обратно на blue

Плюсы: мгновенный switch, instant rollback
Минусы: нужно 2x ресурсов во время switch
```
^dep-blue-green

## A/B Testing

```
Два Deployment'а + два Service'а + Ingress routing

Ingress:
  if Cookie: beta=true → kiada-B service → Deployment B pods (v2)
  else                 → kiada-A service → Deployment A pods (v1)

→ конкретные пользователи видят версию B
→ собираем метрики, сравниваем A vs B
→ зависит от Ingress implementation (не все поддерживают)
```
^dep-ab

## ⚠️ Подводные камни

#### 1. replicas в manifest файле

- `kubectl scale deploy kiada --replicas 5` → потом `kubectl apply -f deploy.yaml` — если **replicas: 3** в файле → **ОТКАТИТ к 3**
- **Совет:** не указывай **replicas** в manifest, скейль через `kubectl scale`
- `kubectl apply edit-last-applied deploy kiada` → убрать **replicas** из **last-applied-configuration**

#### 2. Rolling update + web app

- Браузер получает **HTML от v1**, **CSS от v2** → может **сломать UI**
- **Решение:** **session affinity** или **Blue/Green**

#### 3. Scaling RS напрямую

- **Deployment controller** откатит replicas обратно
- Изменяй **ТОЛЬКО** через **Deployment**

^dep-pitfalls

## Полезные команды

```bash
kubectl get deploy                       # список
kubectl get deploy -o wide               # + containers, images, selector
kubectl describe deploy kiada            # детали + events + conditions
kubectl rollout status deploy kiada      # отслеживание rollout
kubectl rollout history deploy kiada     # история ревизий
kubectl rollout undo deploy kiada        # rollback
kubectl rollout pause/resume deploy kiada
kubectl rollout restart deploy kiada     # restart pod'ов
kubectl set image deploy kiada kiada=luksa/kiada:0.6  # update image
kubectl patch deploy kiada --patch '...' # update несколько полей
kubectl scale deploy kiada --replicas 5  # scaling
```
^dep-commands

## Связь
- [[Deployment — стратегии обновления]] — Recreate vs RollingUpdate, maxSurge/maxUnavailable (Ch.15)
- [[ReplicaSet]] — Deployment управляет RS'ами (Ch.14)
- [[Ingress]] — A/B routing через Ingress (Ch.12)
- [[Gateway API]] — traffic splitting и mirroring через HTTPRoute (Ch.13)
- [[StatefulSet — концепт]] — для stateful workloads вместо Deployment (Ch.16)
