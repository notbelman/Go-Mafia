- **Pod'ы эфемерны** — при пересоздании получают новый IP. Клиент не может хардкодить IP Pod'а. **Service** решает это: стабильный IP + DNS имя + load balancing перед группой Pod'ов. Pod'ы приходят и уходят, Service остаётся. Связь с Pod'ами через **label selector**
- Типы: **ClusterIP** (внутри кластера), **NodePort** (ClusterIP + порт на каждой ноде), **LoadBalancer** (NodePort + внешний LB), **ExternalName** (CNAME alias)
- Pod'ы общаются через **flat NAT-less network** (все pod'ы в одной L3 сети, видят реальные IP друг друга) — каждый pod видит каждый pod напрямую, без NAT, независимо от ноды
- **externalTrafficPolicy:** Cluster (default — равномерно, но extra network hop — трафик может уйти на pod на другой ноде + source IP теряется) vs Local (без hop, сохраняет source IP, но неравномерное распределение)
- **internalTrafficPolicy: Local** — трафик только к pod'ам на той же ноде. Topology-aware hints — мягкая версия: предпочитать pod'ы в той же зоне, но не ограничиваться ими

---

## Зачем Service

#### Проблема

- **Pod'ы эфемерны** → IP меняется при пересоздании
- **Horizontal scaling** → несколько pod'ов, каждый со своим IP
- Клиент **не может хардкодить IP** pod'а

#### Service решает

- **Стабильный IP** (**ClusterIP**) + **DNS имя**
- **Load balancing** между pod'ами
- Pod'ы могут появляться/исчезать — **Service IP не меняется**

```
        ┌──────────┐
Client─▶│  Service │──▶ Pod A
        │ 10.96.x.x│──▶ Pod B
        └──────────┘──▶ Pod C
```
^svc-why

## Label Selector связывает Service с Pod'ами

```yaml
# Service:
spec:
  selector:
    app: quote        # ← все pod'ы с этим label
  ports:
  - port: 80          # порт Service
    targetPort: 80    # порт на pod'е

# Pod:
metadata:
  labels:
    app: quote        # ← матчится с selector'ом Service
    rel: stable       # ← Service не смотрит на этот label
```
^svc-selector


## Типы Service

| Тип | Доступность | Use case |
|---|---|---|
| **ClusterIP** | Внутри кластера | Backend сервисы |
| **NodePort** | ClusterIP + nodePort на каждой ноде (30000-32767) | Dev/test, bare metal |
| **LoadBalancer** | NodePort + внешний LB | Production (cloud) |
| **ExternalName** | CNAME в DNS | Alias для внешних сервисов |
^svc-types

### ClusterIP (default)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: quote
spec:
  type: ClusterIP              # default, можно не указывать
  selector:
    app: quote
  ports:
  - name: http
    port: 80                   # порт Service
    targetPort: 80             # порт на Pod'е
    protocol: TCP
```
^svc-clusterip

### NodePort

```yaml
spec:
  type: NodePort
  selector:
    app: kiada
  ports:
  - name: http
    port: 80                   # ClusterIP порт
    targetPort: 8080           # порт на Pod'е
    nodePort: 30080            # порт на КАЖДОЙ ноде (30000-32767)
  - name: https
    port: 443
    targetPort: 8443
    nodePort: 30443
```
^svc-nodeport


**Клиент** → `<любая_нода_IP>:30080` → **Service** → **Pod** (на любой ноде)

```
  Node A (:30080) ──┐
                    ├──▶ Service ──▶ Pod (может быть на Node A или B)
  Node B (:30080) ──┘
```

> **Не важно** на какую ноду пришёл запрос — K8s форвардит на **любой pod**

^svc-nodeport-flow

### LoadBalancer

```yaml
spec:
  type: LoadBalancer           # = NodePort + внешний LB
  selector:
    app: kiada
  ports:
  - name: http
    port: 80
    targetPort: 8080
    nodePort: 30080            # опционально (можно опустить)

# После создания:
# kubectl get svc kiada
# EXTERNAL-IP: 34.56.78.90    ← IP балансировщика
```
^svc-loadbalancer


**Client** → **Load Balancer** (`34.56.78.90:80`) → **Node**`:30080` → **Service** → **Pod**

> Если кластер **не поддерживает LB** → `EXTERNAL-IP = <pending>`
> **Bare metal** → установить **MetalLB**

^svc-lb-flow

### ExternalName

```yaml
spec:
  type: ExternalName
  externalName: worldtimeapi.org    # CNAME record в DNS
```

- **Нет ClusterIP**, нет endpoints
- Просто **DNS alias**: `time-api.ns.svc.cluster.local` → `worldtimeapi.org`
- **Полезно**: мигрировать внешний сервис под K8s **без изменения клиентов**

^svc-externalname

## External Traffic Policy

#### externalTrafficPolicy: Cluster (default)

- ✅ **Равномерное распределение** между pod'ами
- ❌ **Extra network hop** (нода → другая нода)
- ❌ **Source IP** заменяется на IP ноды (**SNAT**)

#### externalTrafficPolicy: Local

- ✅ **Нет лишних hop'ов**
- ✅ **Source IP** клиента **сохраняется**
- ❌ **Неравномерное распределение** (нода с 1 pod'ом получает столько же трафика как нода с 3)
- ❌ Если на ноде **нет pod'ов** → `connection refused` (решение: **healthCheckNodePort** для LB)

^svc-external-traffic

```
Cluster policy:                    Local policy:
  Node A (1 pod) ─── 33%            Node A (1 pod) ─── 50%
  LB ─▶                             LB ─▶
  Node B (2 pods) ── 33% each       Node B (2 pods) ── 25% each
```
^svc-traffic-diagram


## Internal Traffic Policy

```yaml
spec:
  internalTrafficPolicy: Local     # трафик только к pod'ам на той же ноде
```

- Если на ноде **нет pod'ов** сервиса → `connection refused`
- **Use case**: per-node daemon'ы, **node-local agents**

^svc-internal-traffic

## Session Affinity

```yaml
spec:
  sessionAffinity: ClientIP        # все соединения от одного IP → один pod
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 10800       # 3 часа default
```

> Только **None** или **ClientIP** (нет cookie-based — Service работает на **L4**, не **L7**)

^svc-session-affinity

## Полезные команды

```bash
kubectl get svc                        # список сервисов
kubectl get svc -o wide                # + selector
kubectl expose pod quiz --name quiz    # создать Service из pod'а
kubectl set selector svc quiz app=quiz # изменить selector
kubectl edit svc quiz                  # редактировать Service
kubectl port-forward svc/kiada 8080    # проксировать (для dev)
```
^svc-commands


## Связь
- [[Service — DNS, endpoints, readiness]] — DNS, Endpoints, headless, readiness probes (Ch.11)
- [[Namespaces, labels, selectors, annotations]] — label selector связывает Service с Pod'ами (Ch.7)
- [[Ingress]] — HTTP routing поверх Service'ов (Ch.12)
- [[Pod — что это и зачем]] — pod'ы как backend для Service (Ch.5)
- [[Архитектура кластера]] — kube-proxy реализует Service на каждой ноде (Ch.1)
