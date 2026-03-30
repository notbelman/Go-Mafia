- **HPA** (Horizontal Pod Autoscaler) — автоматически меняет replicas Deployment/StatefulSet по метрикам (CPU, memory, custom). Основной инструмент масштабирования
- **VPA** (Vertical Pod Autoscaler) — автоматически подбирает requests/limits для контейнеров. Требует restart pod'а. Не совместим с HPA по тем же метрикам
- **Cluster Autoscaler** — добавляет/удаляет НОДЫ в кластере. Работает с cloud provider API. Реагирует на Pending pod'ы (нет ресурсов) и недозагруженные ноды
- **Metrics Server** — собирает CPU/memory с kubelet'ов. Нужен для HPA/VPA. Для custom metrics — Prometheus Adapter или KEDA

---

## HPA (Horizontal Pod Autoscaler)

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: kiada
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: kiada
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource                   # встроенные CPU/memory метрики
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70       # target: 70% от requests
  - type: Resource
    resource:
      name: memory
      target:
        type: AverageValue
        averageValue: 500Mi
  behavior:                          # настройка скорости scaling'а
    scaleUp:
      stabilizationWindowSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 300  # 5 мин перед scale down
```
^hpa-manifest

```
Как работает:
  1. HPA controller проверяет метрики каждые 15s (--horizontal-pod-autoscaler-sync-period)
  2. Вычисляет desired replicas: ceil(current × (currentMetric / targetMetric))
  3. Обновляет replicas в Deployment/StatefulSet

Пример:
  current replicas: 3, current CPU: 90%, target CPU: 70%
  desired = ceil(3 × (90/70)) = ceil(3.86) = 4

Типы метрик:
  Resource      — CPU, memory (из Metrics Server)
  Pods          — custom метрики на pod (requests/sec, queue length)
  Object        — метрики K8s объекта (Ingress requests, Service latency)
  External      — внешние метрики (SQS queue depth, Pub/Sub messages)

kubectl get hpa
kubectl describe hpa kiada
kubectl top pods                   # текущее потребление (нужен Metrics Server)
kubectl top nodes
```
^hpa-how-it-works

## Custom Metrics для HPA

```
Metrics Server → только CPU/memory
Для custom metrics нужен metrics adapter:

Prometheus Adapter:
  → собирает метрики из Prometheus
  → регистрирует custom.metrics.k8s.io API
  → HPA читает через этот API

KEDA (Kubernetes Event-Driven Autoscaling):
  → отдельный controller, не HPA (но может управлять HPA)
  → поддерживает 50+ event sources из коробки
  → Kafka, RabbitMQ, Redis, PostgreSQL, AWS SQS, Azure Queue...
  → scale to zero (HPA не умеет: minReplicas ≥ 1)

Пример HPA с custom metric:
  metrics:
  - type: Pods
    pods:
      metric:
        name: http_requests_per_second
      target:
        type: AverageValue
        averageValue: "100"          # target: 100 rps на pod
```
^hpa-custom-metrics

## VPA (Vertical Pod Autoscaler)

```
Не встроен в K8s — нужно установить отдельно.

Три компонента:
  Recommender   — анализирует потребление, рекомендует requests/limits
  Updater       — evict'ит pod'ы для применения рекомендаций
  Admission Controller — устанавливает requests при создании pod'а

Режимы (updatePolicy.updateMode):
  Off           — только рекомендации (kubectl describe vpa)
  Initial       — устанавливает requests только при создании pod'а
  Recreate      — evict'ит pod'ы для обновления requests
  Auto          — = Recreate (in-place update в будущем)

⚠️ VPA + HPA по CPU/memory одновременно = конфликт
   → используй VPA для requests + HPA по custom metrics
   → или используй Multidimensional Pod Autoscaler (MPA)
```
^vpa

## Cluster Autoscaler

```
Работает на уровне НОД (не pod'ов):

Scale UP:
  1. Pod в статусе Pending (не хватает ресурсов на нодах)
  2. Cluster Autoscaler видит Pending pod
  3. Запрашивает новую ноду у cloud provider (ASG, MIG, Node Pool)
  4. Нода создаётся, pod scheduled

Scale DOWN:
  1. Нода недозагружена (< 50% utilization по requests)
  2. Все pod'ы на ноде могут быть перемещены на другие ноды
  3. Нода помечается для удаления
  4. Pod'ы evict'ятся, нода удаляется

⚠️ Не scale down если на ноде:
   → pod без controller'а (не Deployment/RS/STS)
   → pod с local storage (emptyDir с данными)
   → pod с PodDisruptionBudget блокирующий eviction
   → pod с annotation cluster-autoscaler.kubernetes.io/safe-to-evict: "false"

Работает ТОЛЬКО с cloud providers:
  GKE, EKS, AKS, и т.д.
  Bare metal → Karpenter (AWS) или другие решения
```
^cluster-autoscaler

## Взаимодействие HPA + CA

```
                    ┌──────────────────┐
  Metrics Server ──▶│  HPA Controller  │──▶ изменяет replicas
                    └──────────────────┘    в Deployment
                                            │
                                            ▼
                                      Pod Pending?
                                      (нет ресурсов)
                                            │
                    ┌──────────────────┐     ▼
  Cloud Provider ◀──│ Cluster Autoscaler│◀── да → добавить ноду
                    └──────────────────┘

Flow:
  1. Нагрузка растёт → HPA увеличивает replicas
  2. Новые pod'ы Pending (нет места) → CA добавляет ноды
  3. Нагрузка падает → HPA уменьшает replicas
  4. Ноды пустеют → CA удаляет ноды

VPA вписывается отдельно — меняет requests/limits, не replicas
```
^autoscaling-interaction

## Связь
- [[Requests, Limits, QoS]] — HPA скейлит по utilization от requests (Ch.-)
- [[Deployment — стратегии обновления]] — HPA.scaleTargetRef → Deployment (Ch.15)
- [[StatefulSet — концепт]] — HPA может скейлить StatefulSet (Ch.16)
- [[Архитектура кластера]] — Metrics Server как add-on, scheduler для placement (Ch.1)
