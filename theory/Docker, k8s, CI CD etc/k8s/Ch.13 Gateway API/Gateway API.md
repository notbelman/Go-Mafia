- **Gateway API** — следующее поколение после Ingress. Отдельные объекты для gateway и routing. Поддерживает HTTP, TLS, TCP, UDP, gRPC (не только HTTP как Ingress)
- Объекты: **GatewayClass** (провайдер), **Gateway** (точка входа, listeners), **Route** (HTTPRoute/TLSRoute/TCPRoute/UDPRoute/GRPCRoute → связывает Gateway с Service)
- Преимущества: разделение ролей (admin → Gateway, dev → Route), cross-namespace sharing, больше route types, стандартизация фич (не через annotations)
- **HTTPRoute** — routing по host/path/method/headers/query params, traffic splitting (weights), mirroring, header modification, URL rewrite, redirect
- **TLS:** Terminate (gateway расшифровывает, HTTP к backend) vs Passthrough (TLS до backend, routing только по SNI hostname → TLSRoute)

---

## Gateway API vs Ingress

```
                        Ingress              Gateway API
Routing объект          Ingress              HTTPRoute, TLSRoute, TCPRoute...
Gateway объект          (встроен в Ingress)  Gateway (отдельный)
Протоколы               HTTP only            HTTP, TLS, TCP, UDP, gRPC
Cross-namespace         ❌                    ✅ (Route → Gateway в другом ns)
Доп. конфигурация       Annotations          Стандартные поля + filters
Разделение ролей        ❌ (всё в Ingress)   ✅ (admin: Gateway, dev: Route)
Статус                  Stable               HTTPRoute stable, остальные experimental
```
^gw-vs-ingress

## Архитектура

```
                GatewayClass
                (провайдер: Istio, Nginx, Contour...)
                     │
                     ▼
  Client ──▶  Gateway (listeners: ports, protocols, TLS)
                     │
              ┌──────┼──────┐
              ▼      ▼      ▼
          HTTPRoute TLSRoute TCPRoute  (parentRefs → Gateway)
              │      │      │
              ▼      ▼      ▼
           Service Service Service     (backendRefs → Service)
              │      │      │
              ▼      ▼      ▼
            Pods   Pods   Pods
```
^gw-architecture

## GatewayClass

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: istio
spec:
  controllerName: istio.io/gateway-controller    # какой controller
  description: "The default Istio GatewayClass"
  # parametersRef:                                # опционально: доп. параметры
  #   group: ...
  #   kind: ...
  #   name: ...

kubectl get gatewayclasses
```
^gw-class

## Gateway

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: kiada
spec:
  gatewayClassName: istio              # ссылка на GatewayClass
  listeners:
  - name: http                         # имя listener'а
    port: 80
    protocol: HTTP                     # HTTP, HTTPS, TLS, TCP, UDP
    hostname: '*.example.com'          # опционально: hostname filter
  - name: https
    port: 443
    protocol: HTTPS
    tls:
      mode: Terminate                  # Terminate или Passthrough
      certificateRefs:
      - name: tls-secret               # Secret с сертификатом
    # allowedRoutes:                    # кто может подключать Routes
    #   namespaces:
    #     from: All / Same / Selector

# kubectl get gtw
# При создании Gateway → controller создаёт LoadBalancer Service + proxy Pod
```
^gw-gateway

## HTTPRoute

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: kiada
spec:
  parentRefs:                          # к какому Gateway привязать
  - name: kiada
  hostnames:                           # фильтр по hostname
  - kiada.example.com
  rules:
  - matches:                           # условия matching
    - path:
        type: PathPrefix               # Exact, PathPrefix, RegularExpression
        value: /api
    - method: POST                     # HTTP method
    - headers:                         # HTTP headers
      - type: Exact
        name: Release
        value: canary
    - queryParams:                     # Query parameters
      - type: Exact
        name: version
        value: "2"
    backendRefs:                       # куда отправить
    - name: kiada-stable
      port: 80
      weight: 9                        # 90% трафика
    - name: kiada-canary
      port: 80
      weight: 1                        # 10% трафика
    filters:                           # модификация трафика
    - type: RequestHeaderModifier
      requestHeaderModifier:
        add: [{name: X-Gateway, value: "true"}]
        remove: [X-Internal]
```
^gw-httproute

## HTTPRoute Filters

```
RequestHeaderModifier    — add/set/remove заголовки запроса
ResponseHeaderModifier   — add/set/remove заголовки ответа
URLRewrite               — переписать path (ReplaceFullPath / ReplacePrefixMatch)
                           и/или hostname
RequestRedirect          — redirect клиента (scheme, port, path, statusCode)
                           → backendRefs не нужен
RequestMirror            — отправить копию запроса на другой backend
                           → ответ зеркала отбрасывается, клиент получает основной
ExtensionRef             — implementation-specific filter (custom object)
```
^gw-filters

## Traffic Splitting и Mirroring

```yaml
# Splitting (canary deployment):
rules:
- backendRefs:
  - name: stable
    port: 80
    weight: 9              # 90%
  - name: canary
    port: 80
    weight: 1              # 10%

# Mirroring (shadow traffic):
rules:
- backendRefs:
  - name: stable
    port: 80
  filters:
  - type: RequestMirror
    requestMirror:
      backendRef:
        name: canary
        port: 80           # получает копию, ответ отбрасывается
```
^gw-splitting-mirroring

## TLS

```
Terminate (gateway расшифровывает):
  Gateway listener: protocol: HTTPS, tls.mode: Terminate
  → HTTPRoute для routing (gateway видит HTTP)
  → gateway → backend: plain HTTP

Passthrough (gateway прозрачно пропускает):
  Gateway listener: protocol: TLS, tls.mode: Passthrough
  → TLSRoute для routing (gateway видит ТОЛЬКО hostname через SNI)
  → gateway → backend: encrypted TLS (end-to-end)
  → НЕ может routing по path/headers (всё зашифровано)
```
^gw-tls

## Другие Route types

```yaml
# TLSRoute (для TLS passthrough):
apiVersion: gateway.networking.k8s.io/v1alpha2
kind: TLSRoute
spec:
  parentRefs: [{name: my-gateway}]
  hostnames: [kiada.example.com]       # routing только по hostname (SNI)
  rules:
  - backendRefs: [{name: kiada, port: 443}]

# TCPRoute:
kind: TCPRoute
spec:
  parentRefs: [{name: my-gateway}]
  rules:
  - backendRefs: [{name: my-tcp-svc, port: 2000}]

# UDPRoute:
kind: UDPRoute                         # аналогично TCPRoute

# GRPCRoute:
kind: GRPCRoute
spec:
  rules:
  - matches:
    - method:
        service: mypackage.MyService
        method: MyMethod
        type: Exact
    backendRefs: [{name: grpc-svc, port: 9000}]
```
^gw-other-routes

## Cross-Namespace

```
Route → Gateway (в другом namespace):
  Gateway listener: allowedRoutes.namespaces.from: All / Selector
  Route parentRefs: name + namespace

Route → Service (в другом namespace):
  Нужен ReferenceGrant в namespace Service'а:

  apiVersion: gateway.networking.k8s.io/v1beta1
  kind: ReferenceGrant
  metadata:
    namespace: service-namespace        # в namespace referent'а
  spec:
    from:                               # кто может ссылаться
    - group: gateway.networking.k8s.io
      kind: HTTPRoute
      namespace: kiada
    to:                                 # на что можно ссылаться
    - group: ''
      kind: Service
      name: some-service               # опционально (без name = любой Service)
```
^gw-cross-namespace

## GAMMA (Service Mesh)

```
Gateway API может управлять не только north/south (внешний → кластер),
но и east/west (сервис → сервис) трафиком.

HTTPRoute с parentRefs → Service (вместо Gateway):
  parentRefs:
  - name: destination-service
    kind: Service
    group: core

→ правила применяются к трафику МЕЖДУ сервисами
→ фундамент для service mesh через Gateway API
```
^gw-gamma

## Связь
- [[Ingress]] — предшественник Gateway API, L7 only (Ch.12)
- [[Service — типы и routing]] — Gateway API работает поверх ClusterIP Services (Ch.11)
- [[Secrets и Downward API]] — TLS Secret для сертификатов gateway (Ch.8)
- [[Namespaces, labels, selectors, annotations]] — cross-namespace через ReferenceGrant (Ch.7)
