- Библиотека: **google.golang.org/grpc** (grpc-go). Сервер: `grpc.NewServer()`. Клиент: `grpc.Dial()` / `grpc.NewClient()` (Go 1.22+)
- **Стандартные балансировщики** grpc-go: `pick_first` (дефолт, один backend), `round_robin` (по кругу). Для продакшна: **Headless Service** (K8s) + `round_robin` или **L7 proxy** (Envoy)
- **Reflection** (`grpc.reflection.Register`) — сервер отдаёт свою схему. Позволяет `grpcurl` / Evans работать **без proto файлов**. Включать в dev, выключать в проде (информация о API)
- **Профилирование**: `grpc_prometheus` (метрики latency/errors per method), Go pprof (CPU/memory), `opentelemetry-go` (distributed tracing)

---

## Библиотека и базовый сервер

```go
import (
    "google.golang.org/grpc"
    pb "myapp/gen/order/v1"
)

type server struct {
    pb.UnimplementedOrderServiceServer  // forward compatibility
}

func main() {
    lis, _ := net.Listen("tcp", ":50051")

    s := grpc.NewServer(
        grpc.ChainUnaryInterceptor(
            logging, metrics, auth, recovery,
        ),
    )
    pb.RegisterOrderServiceServer(s, &server{})

    // Reflection (для grpcurl в dev)
    reflection.Register(s)

    s.Serve(lis)
}
```

**UnimplementedXxxServer**: обязательно встраивать. Если добавить RPC в proto → сервер без этого embedding **не скомпилируется**. С embedding → новый RPC вернёт `UNIMPLEMENTED` по дефолту.

## Клиент

```go
// grpc.Dial — deprecated с Go 1.22
// grpc.NewClient — новый способ
conn, err := grpc.NewClient(
    "dns:///myservice.default.svc.cluster.local:50051",
    grpc.WithTransportCredentials(insecure.NewCredentials()),
    grpc.WithDefaultServiceConfig(`{"loadBalancingConfig": [{"round_robin":{}}]}`),
)
defer conn.Close()

client := pb.NewOrderServiceClient(conn)

ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

resp, err := client.CreateOrder(ctx, &pb.CreateOrderRequest{...})
```

## Балансировка в grpc-go

### Встроенные балансировщики

| Policy | Как работает | Когда |
|:--|:--|:--|
| `pick_first` (дефолт) | Подключается к **первому** resolved IP, остальные — fallback | Один backend, простой случай |
| `round_robin` | По кругу между всеми resolved IP | Headless Service в K8s |

```go
// round_robin через service config
grpc.WithDefaultServiceConfig(`{"loadBalancingConfig": [{"round_robin":{}}]}`)
```

### С Kubernetes Headless Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myservice
spec:
  clusterIP: None  # headless!
  selector:
    app: myservice
```

DNS `myservice.default.svc.cluster.local` → возвращает **IP всех подов** (не ClusterIP).

grpc-go с `round_robin` → подключается ко всем подам → распределяет запросы.

**Проблема**: DNS кэш. grpc-go по дефолту re-resolve'ит DNS редко. Новые поды → клиент не знает. Решение: `dns:///` scheme (встроенный DNS resolver с периодическим refresh) или `MaxConnectionAge` на сервере.

### Когда встроенного мало

- **Weighted round robin**: нужен кастомный balancer или Envoy
- **Least connections**: нет в grpc-go, нужен L7 proxy
- **По зонам/регионам**: service mesh (Istio)

## Reflection

```go
import "google.golang.org/grpc/reflection"

reflection.Register(s)  // сервер отдаёт свою схему
```

```bash
# grpcurl без proto файлов (reflection)
grpcurl -plaintext localhost:50051 list
grpcurl -plaintext localhost:50051 describe order.v1.OrderService
grpcurl -plaintext -d '{"id": "123"}' localhost:50051 order.v1.OrderService/GetOrder
```

**Evans** — интерактивный gRPC-клиент (REPL):
```bash
evans --host localhost --port 50051 -r  # -r = reflection mode
```

**В проде**: выключить reflection (утечка API-структуры). Или ограничить доступ через interceptor.

## Профилирование и мониторинг

### grpc_prometheus (метрики)

```go
import grpc_prometheus "github.com/grpc-ecosystem/go-grpc-prometheus"

s := grpc.NewServer(
    grpc.ChainUnaryInterceptor(grpc_prometheus.UnaryServerInterceptor),
    grpc.ChainStreamInterceptor(grpc_prometheus.StreamServerInterceptor),
)
grpc_prometheus.Register(s)
grpc_prometheus.EnableHandlingTimeHistogram()  // latency per method
```

Метрики: `grpc_server_handled_total` (count по code+method), `grpc_server_handling_seconds` (latency histogram).

### Go pprof

```go
import _ "net/http/pprof"
go http.ListenAndServe(":6060", nil)  // отдельный HTTP порт для pprof
```

```bash
go tool pprof http://localhost:6060/debug/pprof/profile?seconds=30
```

### OpenTelemetry (distributed tracing)

```go
import "go.opentelemetry.io/contrib/instrumentation/google.golang.org/grpc/otelgrpc"

s := grpc.NewServer(
    grpc.StatsHandler(otelgrpc.NewServerHandler()),
)
```

Trace propagation через metadata — trace_id прокидывается между сервисами автоматически.

## Graceful Shutdown

```go
sigCh := make(chan os.Signal, 1)
signal.Notify(sigCh, syscall.SIGTERM, syscall.SIGINT)

<-sigCh
// 1. Перестать принимать новые соединения
// 2. Дождаться завершения текущих RPC
s.GracefulStop()

// Или с таймаутом:
go func() {
    time.Sleep(30 * time.Second)
    s.Stop()  // жёсткий stop если graceful не успел
}()
s.GracefulStop()
```

## Типичные ошибки

- **Забыл deadline**: `client.GetUser(ctx, req)` с `context.Background()` → горутина может висеть вечно
- **DefaultClient без настроек**: `grpc.Dial("addr")` без interceptors, без keepalive, без балансировки
- **Не закрыл conn**: `conn, _ := grpc.Dial(...)` без `defer conn.Close()` → утечка соединений
- **Один conn на всё**: это ОК! HTTP/2 мультиплексирует. Один `grpc.ClientConn` на сервис = норма

## Связь
- [[gRPC — что это и когда]] — когда gRPC, когда REST
- [[Interceptors]] — middleware для gRPC
- [[Deadlines и Timeouts]] — каждый вызов с deadline
- [[Load Balancing]] — L4 vs L7, client-side
- [[Health Checking]] — readiness/liveness в K8s
