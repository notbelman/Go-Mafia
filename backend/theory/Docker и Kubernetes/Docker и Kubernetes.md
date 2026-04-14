## Быстрая навигация

- [[Docker]] — образы, контейнеры, слои, Compose, Dockerfile
- [[k8s]] — полная карта Kubernetes по темам

---

## [[Docker]]

#### [[Что это]]
Что такое Docker, зачем нужен, контейнеры vs VM

#### [[Docker Image]]
Образы, registry, тэги, pull/push, базовые образы

#### [[Слои Docker]]
Union filesystem, слои образа, copy-on-write, кэш слоёв

#### [[Dockerfile]]
Инструкции FROM, RUN, COPY, ENV, ENTRYPOINT, CMD, best practices

#### [[Multi-stage build для Go]]
Многоэтапная сборка, уменьшение размера образа, scratch образ

#### [[Docker Container]]
Жизненный цикл контейнера, run/stop/rm, networking, ресурсы

#### [[Docker Volumes]]
Volumes vs bind mounts vs tmpfs, когда что использовать

#### [[Docker Compose]]
Compose файл, services/networks/volumes, зависимости, override

#### [[Docker на Mac-Windows. Почему VM не вымерли]]
Почему Docker на Mac/Windows требует VM, Lima, Colima, Rosetta

---

## [[k8s]]

### Основы и архитектура
- [[Kubernetes — обзор (Kubernetes in Action)]]
- [[Архитектура кластера]]
- [[Контейнеры — что внутри]]
- [[Kubernetes API и манифесты]]

### Pods
- [[Pod — что это и зачем]]
- [[Multi-container Pods]]
- [[Pod lifecycle]]
- [[Probes и Lifecycle Hooks]]

### Организация и конфигурация
- [[Namespaces, labels, selectors, annotations]]
- [[ConfigMaps]]
- [[Secrets и Downward API]]

### Сеть и трафик
- [[Service — типы и routing]]
- [[Service — DNS, endpoints, readiness]]
- [[Ingress]]
- [[Gateway API]]

### Storage
- [[PV, PVC, StorageClass]]

### Workloads
- [[ReplicaSet]]
- [[Deployment — стратегии обновления]]
- [[Deployment — rollback и стратегии]]
- [[StatefulSet — концепт]]
- [[StatefulSet — updates и Operators]]

### Продвинутое
- [[Requests, Limits, QoS]]
- [[HPA, VPA, Cluster Autoscaler]]
- [[Affinity, Taints, Tolerations]]
- [[Admission Controllers и Webhooks]]
