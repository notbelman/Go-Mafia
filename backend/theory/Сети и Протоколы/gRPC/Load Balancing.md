- **Проблема**: HTTP/2 = одно TCP-соединение, все запросы мультиплексируются. L4 балансирует **соединения**, не запросы → все запросы на один pod
- **Решения**: **L7 proxy** (Envoy — разбирает HTTP/2, каждый RPC отдельно), **client-side** (headless service + round_robin в grpc-go), **service mesh** (Istio sidecar)
- **MaxConnectionAge** — принудительно закрыть соединение → клиент переподключается → DNS re-resolve → **попадает на новые поды**. Без этого: деплой, а трафик на старых

---

## Проблема

```
L4 балансировщик (K8s Service, NLB):
  Client ──TCP──→ Pod A (все 1000 RPC сюда)
                  Pod B (простаивает)
                  Pod C (простаивает)
```

HTTP/2 = одно долгоживущее TCP-соединение. L4 балансирует при `connect()`, не при каждом RPC.

## Решения

**L7 proxy** (Envoy, Istio, nginx `grpc_pass`):
- Разбирает HTTP/2, каждый RPC маршрутизирует отдельно
- Envoy — стандарт де-факто для gRPC

**Client-side** (grpc-go):
- Headless Service в K8s → DNS возвращает IP всех подов
- `round_robin` policy
- Минус: DNS кэш, нужен re-resolve при изменении подов

**Service mesh** (Istio):
- Envoy sidecar в каждом поде, L7 прозрачно
- Плюс: tracing, circuit breaker, mTLS "бесплатно"
- Минус: сложность, ~20% overhead

## MaxConnectionAge и деплой

После деплоя клиенты держат соединения к старым подам. `MaxConnectionAge` принудительно закрывает → клиент переподключается → DNS re-resolve → новые поды.

## Что выбрать

Мало сервисов → client-side + headless. Продакшн с десятками сервисов → L7 proxy или mesh.

## Связь
- [[gRPC в Go — практика]] — pick_first, round_robin, headless service
- [[Keepalive]] — MaxConnectionAge для ротации
- [[gRPC — что это и когда]] — HTTP/2 мультиплексинг = корень проблемы
