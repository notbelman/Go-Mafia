- **Ingress** = L7 (HTTP) reverse proxy перед Service'ами. Один IP для множества сервисов. Routing по Host header и URL path
- Три компонента: **Ingress object** (правила routing), **Ingress controller** (следит за API, конфигурирует proxy), **Reverse proxy** (Nginx/Envoy/HAProxy — обрабатывает трафик)
- Правила: host-based (`kiada.example.com`) + path-based (`/quote`, `/questions`). pathType: Exact, Prefix (по элементам пути), ImplementationSpecific
- **TLS termination:** proxy терминирует HTTPS, форвардит HTTP на pod. Сертификат + ключ в Secret (type: tls), ссылка в `spec.tls`
- **IngressClass** — какой controller обрабатывает Ingress. Один кластер может иметь несколько controllers. Default class через annotation

---

## Ingress vs LoadBalancer Service

```
LoadBalancer Service:                 Ingress:
  Каждый сервис = свой public IP       Один public IP для всех сервисов
  L4 (TCP/UDP)                         L7 (HTTP/HTTPS)
  Нет routing по path/host             Routing по Host + path
  Нет TLS termination                  TLS termination встроен
  Нет cookie session affinity          Cookie session affinity возможен
  Нет URL rewriting                    URL rewriting возможен

→ Для HTTP сервисов Ingress почти всегда лучше
```
^ing-vs-lb

## Как работает

```
1. Client → DNS lookup (kiada.example.com → Ingress IP)
2. Client → HTTP request → Ingress proxy (Nginx/Envoy)
3. Proxy смотрит Host header + path → выбирает backend
4. Proxy → HTTP request → Pod IP напрямую (не через Service ClusterIP)

        DNS: kiada.example.com → 34.56.78.90

        Client ──HTTPS──▶ Ingress Proxy ──HTTP──▶ Pod
                          (TLS termination)
                          Host: kiada.example.com → kiada service pods
                          Host: api.example.com
                            /quote → quote service pods
                            /questions → quiz service pods
```
^ing-flow

## Ingress Controller

```
Controller = software component (Nginx, Traefik, Ambassador, Contour, GLBC)

  Watches: Ingress, Service, EndpointSlice objects
  Configures: reverse proxy (Nginx config, Envoy config)
  Exposes: proxy через LoadBalancer Service (обычно)

Популярные:
  kubernetes/ingress-nginx  — community Nginx controller
  Traefik                   — built-in Let's Encrypt
  Ambassador/Emissary       — Envoy-based
  Contour                   — Envoy-based
  Cloud-specific: GLBC (GKE), ALB (AWS), AGIC (Azure)
```
^ing-controller

## Manifest

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: kiada
spec:
  ingressClassName: nginx              # какой controller (опционально)
  tls:                                 # TLS termination
  - secretName: tls-example-com
    hosts: ["*.example.com"]
  defaultBackend:                      # catch-all (если ни одно правило не match)
    service:
      name: fun404
      port:
        name: http
  rules:
  - host: kiada.example.com           # правило 1: host-based
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: kiada
            port:
              name: http
  - host: api.example.com             # правило 2: path-based routing
    http:
      paths:
      - path: /quote
        pathType: Exact
        backend:
          service:
            name: quote
            port:
              name: http
      - path: /questions
        pathType: Prefix
        backend:
          service:
            name: quiz
            port:
              name: http
```
^ing-manifest

## Path Matching

```
pathType: Exact
  /foo  → matches /foo
  /foo  → NOT /foo/ , /foo/bar , /FOO

pathType: Prefix
  /foo  → matches /foo , /foo/ , /foo/bar
  /foo  → NOT /foobar (split по / , сравнение по элементам)
  /     → matches всё

pathType: ImplementationSpecific
  → зависит от controller (например, wildcards в GKE)

Приоритет: Exact > Prefix (длинный > короткий)
```
^ing-path-matching

## Host Matching

```
kiada.example.com    → exact match
*.example.com        → matches kiada.example.com, api.example.com
                     → NOT example.com, foo.bar.example.com
                     → wildcard покрывает ОДИН элемент DNS

Без host → match всех хостов
Exact host > wildcard
```
^ing-host-matching

## TLS

```yaml
spec:
  tls:
  - secretName: tls-example-com       # Secret type: kubernetes.io/tls
    hosts:                             # hosts ДОЛЖНЫ совпадать с сертификатом
    - "*.example.com"

# Secret:
kubectl create secret tls tls-example-com \
  --cert=server.crt --key=server.key

TLS passthrough (end-to-end encryption):
  → нестандартная фича, зависит от controller
  → Nginx: annotation nginx.ingress.kubernetes.io/ssl-passthrough: "true"

TLS termination (стандарт):
  → proxy терминирует TLS
  → proxy → pod: plain HTTP
  → pod'у не нужно знать про HTTPS
```
^ing-tls

## IngressClass

```yaml
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: nginx
  annotations:
    ingressclass.kubernetes.io/is-default-class: "true"  # default class
spec:
  controller: k8s.io/ingress-nginx     # какой controller
  # parameters:                         # опционально: ссылка на доп. конфиг
  #   apiGroup: ...
  #   kind: ...
  #   name: ...
```
^ing-class

```
kubectl get ingressclasses             # доступные классы
Ingress без ingressClassName → используется default IngressClass
Несколько controllers → каждый Ingress указывает свой class
```
^ing-class-usage

## Дополнительная конфигурация через Annotations

```yaml
# Cookie session affinity (Nginx):
metadata:
  annotations:
    nginx.ingress.kubernetes.io/affinity: cookie
    nginx.ingress.kubernetes.io/session-cookie-name: MY_COOKIE

# Другие annotation-фичи (зависят от controller):
  → URL rewriting
  → HTTP auth (basic/digest)
  → Rate limiting
  → CORS
  → Redirects
  → Custom timeouts
  → Proxy buffer size

→ Стандартный Ingress API минимален
→ Всё остальное через annotations (Nginx) или custom objects (GKE BackendConfig)
```
^ing-annotations

## Полезные команды

```bash
kubectl get ingress                    # или ing
kubectl describe ing kiada
kubectl get ingressclasses

# Доступ к Ingress:
curl --resolve kiada.example.com:80:<INGRESS_IP> http://kiada.example.com
# Или добавить в /etc/hosts:
# <INGRESS_IP> kiada.example.com api.example.com
```
^ing-commands

## Связь
- [[Service — типы и routing]] — Ingress форвардит на ClusterIP Service (Ch.11)
- [[Gateway API]] — следующее поколение Ingress API (Ch.13)
- [[Secrets и Downward API]] — TLS Secret для сертификатов (Ch.8)
- [[Namespaces, labels, selectors, annotations]] — annotations для доп. конфигурации (Ch.7)
