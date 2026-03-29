- **DNS:** pod'ы находят сервисы по имени через CoreDNS. `<svc>` в том же ns, `<svc>.<ns>` между ns, FQDN: `<svc>.<ns>.svc.cluster.local`
- **Endpoints / EndpointSlice:** K8s автоматически создаёт и обновляет список pod IP для каждого Service. EndpointSlice — замена Endpoints (до 100 endpoints на slice, лучше performance)
- **Headless Service** (clusterIP: None) — DNS возвращает IP всех pod'ов вместо ClusterIP. Клиент сам выбирает к какому pod'у подключиться
- **Readiness probe** — "готов ли pod принимать трафик?" Fail → pod убирается из endpoints Service. Контейнер НЕ перезапускается (в отличие от liveness)
- **Service без selector** — endpoints управляются вручную. Use case: проксирование внешних сервисов через K8s DNS

---

## DNS в кластере

```
CoreDNS (или kube-dns) — internal DNS server
Каждый pod автоматически использует его (resolv.conf)

Резолвинг Service по имени:
  quiz                                  → в том же namespace
  quiz.kiada                            → в namespace kiada
  quiz.kiada.svc                        → полнее
  quiz.kiada.svc.cluster.local          → FQDN

Как работает:
  /etc/resolv.conf в pod'е:
    nameserver 10.96.0.10               ← ClusterIP kube-dns Service
    search kiada.svc.cluster.local svc.cluster.local cluster.local
    options ndots:5

  → "quiz" → пробует quiz.kiada.svc.cluster.local → нашёл!
```
^dns-resolution

### DNS записи

```
A / AAAA record:
  quiz.kiada.svc.cluster.local → 10.96.136.190 (ClusterIP)

SRV records (порты сервиса):
  _http._tcp.kiada.kiada.svc.cluster.local → 0 100 80 kiada...
  _https._tcp.kiada.kiada.svc.cluster.local → 0 100 443 kiada...
  → клиент может узнать порты сервиса через SRV lookup

CNAME (ExternalName service):
  time-api.kiada.svc.cluster.local → worldtimeapi.org

⚠️ LoadBalancer service: DNS возвращает только ClusterIP, НЕ external IP
```
^dns-records

### Environment Variables (legacy)

```
K8s добавляет env vars для каждого Service в namespace при старте контейнера:

  QUIZ_SERVICE_HOST=10.96.136.190
  QUIZ_SERVICE_PORT=80
  QUIZ_PORT=tcp://10.96.136.190:80
  ...

⚠️ Создаются только при СТАРТЕ контейнера
  → Service должен существовать ДО создания pod'а
  → или контейнер должен быть перезапущен

⚠️ Слишком много сервисов в namespace → "argument list too long" ошибка
  → spec.enableServiceLinks: false — отключить инъекцию

Сейчас все используют DNS, env vars — legacy
```
^dns-env-vars

## Endpoints и EndpointSlice

```
Service создан с selector → K8s автоматически создаёт:
  Endpoints object (deprecated) — все endpoints в одном объекте
  EndpointSlice objects — endpoints разбиты на slices (max 100 по умолчанию)

kubectl get endpoints kiada       # или ep
kubectl get endpointslices -l kubernetes.io/service-name=kiada

EndpointSlice содержит:
  addresses:  IP pod'ов
  ports:      порты
  conditions: ready (true/false)
  topology:   kubernetes.io/hostname, zone
  targetRef:  Pod name/namespace

K8s обновляет автоматически при:
  → добавлении/удалении pod'а с matching labels
  → изменении readiness status pod'а
```
^ep-endpoints

## Headless Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: quote-headless
spec:
  clusterIP: None              # ← headless
  selector:
    app: quote
  ports:
  - port: 80
    targetPort: 80
```
^headless-manifest

```
Regular Service:                    Headless Service:
  nslookup quote                      nslookup quote-headless
  → 10.96.74.151 (ClusterIP)         → 10.244.2.9  (Pod IP)
                                      → 10.244.2.8  (Pod IP)
                                      → 10.244.2.10 (Pod IP)
                                      → 10.244.1.10 (Pod IP)

Regular: клиент → ClusterIP → kube-proxy → random pod
Headless: клиент → DNS → получает все pod IPs → сам выбирает

Use cases:
  → клиент сам делает load balancing
  → клиент хочет подключиться ко ВСЕМ pod'ам
  → StatefulSet: pod'ы должны знать друг друга (peer discovery)
  → базы данных с client-side routing
```
^headless-vs-regular

## Service без selector (ручные endpoints)

```yaml
# Service без selector:
apiVersion: v1
kind: Service
metadata:
  name: external-service
spec:
  ports:
  - port: 80
---
# Endpoints вручную:
apiVersion: v1
kind: Endpoints
metadata:
  name: external-service       # имя ДОЛЖНО совпадать с Service
subsets:
- addresses:
  - ip: 1.1.1.1
  - ip: 2.2.2.2
  ports:
  - name: http
    port: 88
```
^svc-no-selector

```
Use cases:
  → проксирование внешнего сервиса через K8s DNS
  → миграция: сначала external endpoints → потом добавить selector → pod'ы
  → обратная миграция: убрать selector → ручные endpoints → внешний сервис
  → ClusterIP Service остаётся тот же → клиенты не замечают миграцию
```
^svc-no-selector-usecases

## Readiness Probe

```
"Готов ли pod принимать ТРАФИК?"

  Liveness fail  → RESTART контейнер
  Readiness fail → УБРАТЬ pod из Service endpoints (НЕ restart)

  Pod НЕ ready → не получает трафик от Service
  Pod стал ready → добавляется в endpoints
  Pod удаляется → K8s сам убирает из endpoints (readiness не нужен для этого)
```
^readiness-overview

### Конфигурация

```yaml
readinessProbe:
  httpGet:
    port: 8080
    path: /healthz/ready
  initialDelaySeconds: 10     # первая проверка через 10s
  periodSeconds: 5            # каждые 5s
  timeoutSeconds: 2           # ответ за 2s
  failureThreshold: 3         # 3 фейла → not ready
  successThreshold: 2         # 2 успеха → ready (default: 1)

# Типы: httpGet, tcpSocket, exec — как liveness
```
^readiness-config

### Best Practices

```
✅ Всегда определяй readiness probe (иначе pod сразу получает трафик)
✅ Проверяй внутренние зависимости (БД в том же pod'е)
✅ Для HTTP: минимум GET / — лучше чем ничего
✅ Лучше: dedicated endpoint /healthz/ready с проверками
✅ failureThreshold: 1 для быстрого реагирования

❌ НЕ проверяй внешние зависимости (другие сервисы)
   → transient network issue → все pod'ы not ready → cascading failure
❌ НЕ устанавливай слишком маленький timeout
   → нормальная задержка = probe fail = pod убирается из Service

Readiness vs Liveness:
  Liveness:  "сломался ли контейнер?" → restart
  Readiness: "готов ли принимать запросы?" → убрать из endpoints
  Startup:   "запустился ли?" → пока не пройдёт, liveness не стартует
```
^readiness-best-practices

## Не пингуй Service IP

```
$ ping quiz
PING quiz (10.96.136.190): 56 data bytes
... 100% packet loss

Service IP = виртуальный. Работает только с TCP/UDP + конкретный порт.
ICMP (ping) не работает. Это НЕ баг.
```
^svc-no-ping

## Связь
- [[Service — типы и routing]] — ClusterIP, NodePort, LoadBalancer, traffic policy (Ch.11)
- [[Probes и Lifecycle Hooks]] — liveness vs readiness vs startup (Ch.6)
- [[Ingress]] — HTTP routing поверх Service (Ch.12)
- [[StatefulSet — headless Service]] — peer discovery через headless (Ch.16)
- [[Архитектура кластера]] — CoreDNS как add-on компонент (Ch.1)
