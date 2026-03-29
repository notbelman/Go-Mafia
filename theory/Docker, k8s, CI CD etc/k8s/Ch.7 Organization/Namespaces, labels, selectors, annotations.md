- **Namespace** — виртуальный кластер внутри физического. Scope для имён объектов. Не даёт изоляцию runtime/network (только именование + RBAC)
- **Labels** — key-value пары на объектах для идентификации и группировки. Используются selectors для фильтрации. Ключевой механизм K8s (Service→Pod, ReplicaSet→Pod)
- **Label selectors:** equality-based (`app=kiada`, `rel!=canary`) и set-based (`app in (quiz,quote)`, `!rel`). Комбинируются через запятую (AND)
- **Annotations** — key-value для метаданных, не для фильтрации. До 256KB, любые символы. Для описаний, контактов, build info, инструментов
- **nodeSelector** — простой equality-based selector для scheduling pod'а на ноды с определёнными labels. **nodeAffinity** — более мощный set-based вариант

---

## Namespaces

```
Kubernetes Cluster
┌──────────────────────────────────────────────┐
│  default          kube-system    my-team      │
│  ┌──────────┐    ┌──────────┐  ┌──────────┐  │
│  │ Pod: app  │    │ coredns  │  │ Pod: app  │  │  ← одинаковые имена OK
│  │ Svc: api  │    │ etcd     │  │ Svc: api  │  │
│  └──────────┘    │ kube-dns │  └──────────┘  │
│                  └──────────┘                │
└──────────────────────────────────────────────┘

Имена уникальны ВНУТРИ namespace, не между ними
```
^ns-overview

### Что namespaced, что нет

```
Namespaced (большинство):          Cluster-scoped:
  Pod, Service, Deployment           Node
  ConfigMap, Secret                  PersistentVolume
  PersistentVolumeClaim              StorageClass
  Event, Ingress                     Namespace (сам по себе)

kubectl api-resources → колонка NAMESPACED
```
^ns-scoped

### Работа с namespaces

```bash
kubectl get ns                              # список namespaces
kubectl create ns my-team                   # создать
kubectl get pods -n kube-system             # объекты в namespace
kubectl get pods -A                         # все namespaces (--all-namespaces)
kubectl config set-context --current --namespace my-team  # переключиться
kubectl delete ns my-team                   # удалить namespace + ВСЕ объекты в нём
```
^ns-commands

### Изоляция (точнее, её отсутствие)

```
Namespaces НЕ дают:
  ❌ Runtime изоляцию — pod'ы из разных ns могут быть на одной ноде
  ❌ Network изоляцию — по умолчанию pod'ы из разных ns могут общаться
     (нужен NetworkPolicy для ограничения)
  ❌ Замену разным кластерам для prod/staging/dev

Namespaces ДАЮТ:
  ✅ Scope имён — одинаковые имена в разных ns
  ✅ RBAC scope — права пользователей привязаны к namespace
  ✅ Resource quotas — лимиты CPU/memory на namespace
  ✅ Организацию — команды работают в своих ns
```
^ns-isolation

## Labels

```yaml
metadata:
  labels:
    app: kiada          # какое приложение
    rel: stable         # тип релиза (stable/canary)
    tier: frontend      # уровень (frontend/backend)
    env: production     # окружение
    version: "1.0"      # версия
```
^lb-example

### Работа с labels

```bash
kubectl get pods --show-labels              # показать все labels
kubectl get pods -L app,rel                 # показать конкретные labels как колонки
kubectl label pod my-pod app=kiada          # добавить label
kubectl label pod my-pod app=quote --overwrite  # изменить
kubectl label pod my-pod app-               # удалить (минус в конце)
kubectl label pod --all suite=kiada-suite   # добавить всем pod'ам
```
^lb-commands

### Правила синтаксиса

```
Key:
  [prefix/]name
  prefix: DNS subdomain, ≤253 символов (example.com/)
  name: ≤63 символа, alphanumeric + hyphens/underscores/dots
  kubernetes.io/ и k8s.io/ — зарезервированы

Value:
  ≤63 символа
  alphanumeric + hyphens/underscores/dots
  НЕТ пробелов, спецсимволов
  может быть пустым ("")
```
^lb-syntax

## Label Selectors

```bash
# Equality-based:
kubectl get pods -l app=kiada                 # app равен kiada
kubectl get pods -l app!=kiada                # app НЕ равен kiada

# Set-based:
kubectl get pods -l 'app in (quiz,quote)'     # app = quiz ИЛИ quote
kubectl get pods -l 'app notin (kiada)'       # app ≠ kiada
kubectl get pods -l rel                       # label rel СУЩЕСТВУЕТ
kubectl get pods -l '!rel'                    # label rel НЕ существует

# Комбинация (AND):
kubectl get pods -l app=quote,rel=canary      # оба условия

# Удаление по selector:
kubectl delete pods -l rel=canary             # удалить все canary pod'ы
```
^lb-selectors

### Selectors в манифестах (внутри K8s)

```
Service → выбирает Pod'ы:
  selector:
    app: kiada

ReplicaSet → владеет Pod'ами:
  selector:
    matchLabels:
      app: kiada

Pod → выбирает Node:
  nodeSelector:
    node-role: front-end

→ Labels + Selectors = как K8s связывает объекты между собой
```
^lb-selectors-manifests

## nodeSelector и nodeAffinity

```yaml
# Простой equality-based:
spec:
  nodeSelector:
    node-role: front-end          # pod только на ноды с этим label

# Мощный set-based (nodeAffinity):
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: node-role
            operator: In           # In, NotIn, Exists, DoesNotExist, Lt, Gt
            values: [front-end]
          - key: skip-me
            operator: DoesNotExist
```
^lb-node-selector

## Field Selectors

```bash
# Фильтрация по полям объекта (не labels):
kubectl get pods --field-selector spec.nodeName=worker-1
kubectl get pods --field-selector status.phase!=Running
kubectl get pods --field-selector metadata.namespace=default

# Поддерживаемые поля зависят от типа объекта
# metadata.name и metadata.namespace — всегда поддерживаются
```
^fs-field-selectors

## Annotations

```yaml
metadata:
  annotations:
    created-by: "Marko Luksa <marko@example.com>"
    git-commit: "abc123def"
    build-timestamp: "2024-01-15T10:30:00Z"
    description: "Frontend service for Kiada application"
    contact-phone: "+1 234 567 890"
```
^an-example

```
Labels vs Annotations:

  Labels:                          Annotations:
  ≤63 символа                      ≤256KB
  Для идентификации и фильтрации   Для метаданных, описаний
  Используются selectors           НЕ используются для фильтрации
  Ограниченный набор символов      Любые символы

kubectl annotate pod my-pod description="My app"     # добавить
kubectl annotate pod my-pod description- --overwrite  # изменить
kubectl annotate pod my-pod description-              # удалить
```
^an-vs-labels

### Annotations используемые K8s

```
kubectl.kubernetes.io/last-applied-configuration  — последний applied manifest
kubernetes.io/change-cause                         — причина rollout
ingress.kubernetes.io/rewrite-target               — Ingress конфигурация

Паттерн: новые фичи сначала как annotation → потом как поле в API
```
^an-kubernetes

## Связь
- [[Kubernetes API и манифесты]] — metadata.labels, metadata.annotations (Ch.4)
- [[Service — типы и routing]] — selector связывает Service с Pod'ами (Ch.11)
- [[ReplicaSet]] — selector определяет какие Pod'ы принадлежат ReplicaSet (Ch.14)
- [[Deployment — стратегии обновления]] — labels в pod template (Ch.15)
- [[Pod — что это и зачем]] — nodeSelector для scheduling (Ch.5)
