- **requests** = гарантированный минимум ресурсов. Scheduler использует для placement. **limits** = потолок, превышение → throttling (CPU) или OOM kill (memory)
- **QoS классы:** Guaranteed (requests == limits для всех контейнеров), Burstable (хотя бы один request задан), BestEffort (ничего не задано). При нехватке памяти на ноде — BestEffort убивается первым
- CPU — compressible: превышение limits → throttling (процесс замедляется, НЕ убивается). Memory — incompressible: превышение limits → OOM kill контейнера
- Без requests: scheduler не знает сколько ресурсов нужно pod'у → может перегрузить ноду. Без limits: pod может съесть все ресурсы ноды

---

## Requests vs Limits

```yaml
containers:
- name: app
  image: myapp:1.0
  resources:
    requests:                    # гарантированный минимум
      cpu: 250m                  # 0.25 ядра
      memory: 128Mi              # 128 мебибайт
    limits:                      # максимальный потолок
      cpu: 500m                  # 0.5 ядра
      memory: 256Mi              # 256 мебибайт
```
^res-manifest

```
requests:
  → scheduler использует для выбора ноды (Allocatable - sum(requests) ≥ new request)
  → контейнер ГАРАНТИРОВАННО получает эти ресурсы
  → cgroups: cpu.shares (пропорциональное распределение CPU)

limits:
  → контейнер НЕ МОЖЕТ использовать больше
  → cgroups: cpu.cfs_quota_us (hard cap для CPU), memory.limit_in_bytes
  → превышение CPU → throttling (замедление, НЕ kill)
  → превышение memory → OOM kill контейнера (exit code 137)

⚠️ requests > limits → ошибка валидации
⚠️ limits без requests → requests автоматически = limits
```
^res-how-it-works

## Единицы измерения

```
CPU:
  1    = 1 vCPU / 1 hyperthread
  100m = 0.1 ядра (m = millicores)
  250m = 0.25 ядра

Memory:
  128Mi  = 128 мебибайт (1 Mi = 1024² bytes)
  1Gi    = 1 гибибайт
  128M   = 128 мегабайт (1 M = 1000² bytes) — не путать с Mi!

⚠️ 128Mi ≠ 128M (разница ~5%)
```
^res-units

## QoS классы (Quality of Service)

```
K8s автоматически назначает QoS class каждому pod'у:

Guaranteed:
  → ВСЕ контейнеры: requests == limits (и CPU, и memory)
  → последний на убой при OOM на ноде
  → пример: databases, critical services

Burstable:
  → хотя бы один контейнер имеет requests или limits
  → requests ≠ limits (или заданы не для всех ресурсов)
  → средний приоритет при OOM

BestEffort:
  → НИ ОДИН контейнер не задаёт requests/limits
  → первый на убой при OOM на ноде
  → пример: batch jobs, некритичные задачи

Приоритет убийства при OOM:
  BestEffort → Burstable (по % превышения requests) → Guaranteed
```
^res-qos

```
# QoS = Guaranteed:
resources:
  requests:
    cpu: 500m
    memory: 256Mi
  limits:
    cpu: 500m          # == requests
    memory: 256Mi      # == requests

# QoS = Burstable:
resources:
  requests:
    cpu: 250m
    memory: 128Mi
  limits:
    cpu: 500m          # ≠ requests
    memory: 256Mi

# QoS = BestEffort:
# (нет секции resources вообще)
```
^res-qos-examples

## OOM Kill vs CPU Throttling

```
CPU (compressible resource):
  → pod использует больше limit → CFS throttling
  → процесс ЗАМЕДЛЯЕТСЯ но НЕ убивается
  → метрика: container_cpu_cfs_throttled_periods_total
  → симптом: высокая latency, медленные ответы

Memory (incompressible resource):
  → pod использует больше limit → OOM kill
  → контейнер убивается с exit code 137 (SIGKILL)
  → pod status: OOMKilled
  → restartPolicy определяет что дальше
  → если pod превышает requests но не limits → убьют при давлении на ноде

⚠️ Java: -Xmx должен быть < memory limit (иначе OOM kill)
⚠️ Go: GOMEMLIMIT (с Go 1.19) помогает GC оставаться в рамках
```
^res-oom-throttling

## LimitRange и ResourceQuota

```yaml
# LimitRange — default requests/limits для namespace:
apiVersion: v1
kind: LimitRange
metadata:
  name: defaults
spec:
  limits:
  - default:                     # default limits
      cpu: 500m
      memory: 256Mi
    defaultRequest:              # default requests
      cpu: 100m
      memory: 128Mi
    type: Container

# ResourceQuota — лимит ресурсов на весь namespace:
apiVersion: v1
kind: ResourceQuota
metadata:
  name: quota
spec:
  hard:
    requests.cpu: "10"           # сумма всех requests CPU в ns
    requests.memory: 20Gi
    limits.cpu: "20"
    limits.memory: 40Gi
    pods: "50"                   # макс. pod'ов в namespace
```
^res-limitrange-quota

## Best Practices

```
✅ Всегда задавай requests (scheduler нуждается в них)
✅ Задавай memory limits (защита от OOM всей ноды)
✅ Для production: requests == limits (QoS Guaranteed)
✅ Для dev/staging: Burstable ок (экономия ресурсов)
✅ Используй LimitRange для default'ов в namespace
✅ Используй ResourceQuota для мультитенантности

❌ Не задавай CPU limits слишком низко → throttling → latency
   (спорная рекомендация: многие убирают CPU limits совсем)
❌ Не запускай BestEffort в production
❌ Не путай Mi и M
```
^res-best-practices

## Связь
- [[Контейнеры — что внутри]] — cgroups ограничивают ресурсы контейнера (Ch.2)
- [[Pod — что это и зачем]] — resources в container spec (Ch.5)
- [[Архитектура кластера]] — scheduler использует requests для placement (Ch.1)
- [[HPA, VPA, Cluster Autoscaler]] — автоскейлинг на основе потребления ресурсов
- [[Namespaces, labels, selectors, annotations]] — LimitRange/ResourceQuota per namespace (Ch.7)
