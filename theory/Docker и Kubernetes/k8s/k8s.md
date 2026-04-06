### Основы и архитектура

#### [[Kubernetes — обзор (Kubernetes in Action)]]
Что такое K8s, декларативная модель, зачем нужен, когда не нужен

#### [[Архитектура кластера]]
Control plane (API Server, etcd, Scheduler, Controllers), worker nodes, kubelet, kube-proxy

#### [[Контейнеры — что внутри]]
Linux namespaces, cgroups, image layers, CRI/CSI/CNI интерфейсы

#### [[Kubernetes API и манифесты]]
REST API, структура манифеста (apiVersion/kind/metadata/spec/status), kubectl

---

### Pods

#### [[Pod — что это и зачем]]
Pod как группа контейнеров, shared namespaces, когда объединять, manifest

#### [[Multi-container Pods]]
Sidecar, init containers, native sidecar (K8s 1.28+), порядок запуска

#### [[Pod lifecycle]]
Фазы (Pending/Running/Succeeded/Failed), состояния контейнеров, restart policy

#### [[Probes и Lifecycle Hooks]]
Liveness, startup, readiness probes — httpGet/tcpSocket/exec, postStart, preStop

---

### Организация и конфигурация

#### [[Namespaces, labels, selectors, annotations]]
Namespaces для изоляции, labels для фильтрации, selectors, annotations

#### [[ConfigMaps]]
Создание, использование в pods (env vars и volumes), immutability

#### [[Secrets и Downward API]]
Типы секретов, Base64, security considerations, Downward API для метаданных пода

---

### Сеть и трафик

#### [[Service — типы и routing]]
ClusterIP, NodePort, LoadBalancer, ExternalName, label selectors, session affinity

#### [[Service — DNS, endpoints, readiness]]
CoreDNS, endpoints/endpointslices, headless service, service без selector

#### [[Ingress]]
Ingress vs LoadBalancer, L7 routing (host/path), TLS termination, IngressClass

#### [[Gateway API]]
Gateway API vs Ingress, GatewayClass, HTTPRoute, traffic splitting, cross-namespace

---

### Storage

#### [[PV, PVC, StorageClass]]
PersistentVolume, PersistentVolumeClaim, dynamic provisioning, access modes, reclaim policies

---

### Workloads

#### [[ReplicaSet]]
Spec, reconciliation loop, scaling, pod ownership, обновление template

#### [[Deployment — стратегии обновления]]
Recreate vs RollingUpdate, maxSurge/maxUnavailable, minReadySeconds

#### [[Deployment — rollback и стратегии]]
Rollback, revision history, canary, blue/green, A/B testing

#### [[StatefulSet — концепт]]
Stateful workloads, ordinal naming, headless service, volumeClaimTemplates, at-most-one

#### [[StatefulSet — updates и Operators]]
RollingUpdate, OnDelete, partition для canary, ControllerRevision, Kubernetes Operators

---

### Продвинутое

#### [[Requests, Limits, QoS]]
Requests vs limits, CPU throttling vs OOM kill, QoS классы (Guaranteed/Burstable/BestEffort)

#### [[HPA, VPA, Cluster Autoscaler]]
HPA горизонтальное масштабирование, VPA рекомендации ресурсов, KEDA, Cluster Autoscaler

#### [[Affinity, Taints, Tolerations]]
nodeAffinity, podAffinity/AntiAffinity, taints + tolerations, topologySpreadConstraints

#### [[Admission Controllers и Webhooks]]
Admission pipeline, mutating/validating webhooks, ValidatingAdmissionPolicy, CEL
