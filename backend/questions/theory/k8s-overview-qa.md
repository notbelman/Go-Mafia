---
type: question
topic: Kubernetes
subtopic:
  - Docker
  - Containers
  - Architecture
  - Deployment
  - ReplicaSet
  - Service
  - Ingress
  - Probes
  - HPA
  - CRI
  - Debugging
title: Kubernetes — обзорные вопросы и ответы
---

## Можно ли запустить контейнер на Windows? Можно ли добавить Windows-ноду в Kubernetes?

[[Контейнеры — что внутри#Портабельность|Контейнеры — портабельность]] — образ **не содержит ядро**, зависит от ядра хоста. Windows-контейнеры существуют, но x86 Linux-образ на Windows без эмуляции не запустится. Windows-ноды в K8s поддерживаются начиная с K8s 1.18+ (но control plane всё равно на Linux)

## Как развернуть микросервис в Kubernetes с двумя репликами на разных нодах?

[[Deployment — стратегии обновления]] → указать `replicas: 2` в spec. Для размещения на **разных нодах**: [[Affinity, Taints, Tolerations#Pod Anti-Affinity|Pod Anti-Affinity]] с `topologyKey: kubernetes.io/hostname`

## Почему выбрали Deployment, а не StatefulSet или другие контроллеры?

- [[Deployment — стратегии обновления#Deployment vs ReplicaSet|Deployment vs ReplicaSet]] — Deployment управляет ReplicaSet'ами, даёт **rolling updates и rollback**
- [[StatefulSet — концепт#StatefulSet vs Deployment|StatefulSet vs Deployment]] — StatefulSet нужен только для **stateful** приложений (стабильные имена, persistent storage, упорядоченный запуск)

## Что нужно указать в спецификации Deployment?

[[Deployment — стратегии обновления]] — `replicas`, `selector`, `template` (pod spec). Дополнительно: [[Deployment — rollback и стратегии]] — `strategy` (RollingUpdate/Recreate), `revisionHistoryLimit`, `minReadySeconds`

## Чем отличаются requests и limits для ресурсов?

[[Requests, Limits, QoS#requests|requests]] — **гарантированный минимум**, по нему Scheduler выбирает ноду. [[Requests, Limits, QoS#limits|limits]] — **потолок**, при превышении memory → OOM Kill, CPU → throttling. [[Requests, Limits, QoS#QoS классы|QoS классы]] определяются соотношением requests и limits

## Что такое пробы (probes) в Kubernetes и зачем они нужны?

[[Probes и Lifecycle Hooks]] — три типа:
- **Liveness** — перезапуск при зависании
- **Readiness** — убирает из Service при неготовности
- **Startup** — даёт время на запуск, защищает от преждевременного kill

## Как выставить сервис, чтобы к нему могли обращаться только поды внутри кластера?

[[Service — типы и routing#Типы Service|ClusterIP]] — тип по умолчанию, доступен **только внутри кластера**. DNS: [[Service — DNS, endpoints, readiness#DNS в кластере|CoreDNS]] резолвит `<svc>.<ns>.svc.cluster.local`

## Какие виды масштабирования существуют и как они реализованы в Kubernetes?

[[HPA, VPA, Cluster Autoscaler]]:
- **HPA** — горизонтальное, добавляет pod'ы по метрикам
- **VPA** — вертикальное, меняет requests/limits
- **Cluster Autoscaler** — добавляет/убирает **ноды**

## Как HPA получает кастомные метрики?

[[HPA, VPA, Cluster Autoscaler#Custom Metrics|Custom Metrics]] — через **Prometheus Adapter** (custom.metrics.k8s.io API) или **KEDA** (event-driven autoscaling из внешних источников)

## Как Kubernetes позволяет подключать разные container runtime, storage и network решения?

- [[Контейнеры — что внутри#Container Runtime|CRI]] — Container Runtime Interface (containerd, CRI-O)
- [[PV, PVC, StorageClass]] — CSI драйверы для storage
- [[Архитектура кластера#Add-on компоненты|CNI plugin]] — сетевые плагины (Calico, Cilium, Flannel)

## Опишите процесс создания Deployment: от kubectl apply до запуска подов

[[Архитектура кластера#Как K8s запускает приложение|Как K8s запускает приложение]]:
1. `kubectl` → **API Server** → сохраняет в etcd
2. **Controller** → создаёт ReplicaSet → создаёт Pod'ы
3. **Scheduler** → назначает ноду
4. **Kubelet** → запускает контейнер через CRI
5. **kube-proxy** → настраивает сетевые правила

Подробнее про валидацию: [[Kubernetes API и манифесты]]

## Что происходит на этапе валидации в API server? Можно ли добавить дополнительные проверки?

[[Kubernetes API и манифесты]] — API Server валидирует структуру и поля манифеста. Для кастомных проверок: [[Admission Controllers и Webhooks]] — **Validating/Mutating Webhooks** перехватывают запросы до записи в etcd

## Как отладить distroless контейнер (без shell) в Kubernetes?

[[Pod — что это и зачем#Решение: `kubectl debug`|kubectl debug]] — добавляет **ephemeral debug container** к существующему pod'у без пересоздания. Образ `nicolaka/netshoot` содержит tcpdump, curl, dig, strace

---

## Почему контейнеры (Docker) vs VM — Docker выигрывает

[[Контейнеры — что внутри]] — контейнеры используют **namespace + cgroups** ядра хоста вместо гипервизора. Запуск за секунды, минимальный overhead, одно ядро на все контейнеры

## Что такое Kubernetes

[[Kubernetes — обзор (Kubernetes in Action)]] — система автоматизации деплоя контейнеризированных приложений. **Декларативная модель**: ты описываешь ЧТО, K8s решает КАК

## Deployment в Kubernetes

[[Deployment — стратегии обновления]] — контроллер, управляющий ReplicaSet'ами. Даёт **rolling updates**, **rollback**, **масштабирование**. Подробнее: [[Deployment — rollback и стратегии]]

## Deployment и ReplicaSet

[[Deployment — стратегии обновления#Deployment vs ReplicaSet|Deployment vs ReplicaSet]] — Deployment **владеет** ReplicaSet'ами. При обновлении создаёт новый RS, плавно переводит pod'ы. Напрямую RS почти никогда не создают

## Что такое ReplicaSet в Kubernetes

[[ReplicaSet]] — контроллер, поддерживающий **заданное количество pod'ов**. Reconciliation loop: desired → current state. Выбирает pod'ы через **label selector**

## Агенты на нодах Kubernetes

[[Архитектура кластера#Компоненты подробнее|Worker Node компоненты]]:
- [[Архитектура кластера#Kubelet|Kubelet]] — агент, запускает pod'ы
- [[Архитектура кластера#kube-proxy|kube-proxy]] — сетевые правила
- [[Архитектура кластера#Container Runtime|Container Runtime]] — containerd/CRI-O

## Kubernetes Service — за что отвечает

[[Service — типы и routing]] — стабильный **IP + DNS** для группы pod'ов. Типы: ClusterIP, NodePort, LoadBalancer, ExternalName. [[Service — DNS, endpoints, readiness]] — DNS-резолвинг, endpoints, readiness gates

---

## Что такое Docker и для чего используется Kubernetes?

[[Контейнеры — что внутри]] — Docker пакует приложение в **изолированный контейнер** (namespaces + cgroups). [[Kubernetes — обзор (Kubernetes in Action)]] — оркестрирует контейнеры на **множестве машин**: деплой, масштабирование, self-healing

## Как оцениваете свой опыт в Kubernetes?

> Субъективный вопрос — ответ зависит от кандидата. Ниже ссылки для подготовки

Архитектура: [[Архитектура кластера]]. Основные объекты: [[Pod — что это и зачем]], [[Deployment — стратегии обновления]], [[Service — типы и routing]], [[StatefulSet — концепт]]. Продвинутые темы: [[HPA, VPA, Cluster Autoscaler]], [[Admission Controllers и Webhooks]], [[Affinity, Taints, Tolerations]]

## В чем разница между Pod и Deployment в Kubernetes?

[[Pod — что это и зачем]] — **минимальная единица** деплоя, один или несколько контейнеров. [[Deployment — стратегии обновления]] — **контроллер**, который управляет pod'ами через ReplicaSet. Pod сам себя не перезапустит, Deployment — да

## Что произойдет если Pod управляемый Deployment упадет или будет удален?

[[ReplicaSet]] — reconciliation loop заметит расхождение desired vs current → создаст **новый pod**. [[Pod lifecycle#Restart Policy|restartPolicy]] определяет поведение при крэше контейнера внутри pod'а. [[Архитектура кластера#Controllers|Controller]] постоянно мониторит состояние

## Что такое Ingress в Kubernetes?

[[Ingress]] — объект для маршрутизации **HTTP/HTTPS** трафика внутрь кластера. Правила по host + path → Service. Требует **Ingress Controller** (Nginx, Traefik). Современная альтернатива: [[Gateway API]]
