- Go: два резолвера — **pure Go** (дефолт, кроссплатформенный, использует /etc/resolv.conf напрямую) и **CGO** (через libc, поддерживает mDNS и сложный nsswitch.conf). Переключение: `GODEBUG=netdns=go` или `cgo`
- **DNS-based load balancing**: несколько A-записей → клиент выбирает рандомно. Проблемы: клиент **кэширует** и ходит на один IP, TTL не гарантирует свежесть
- **Service discovery**: SRV-записи (`_grpc._tcp.myservice → host:port`), Consul DNS interface, Kubernetes CoreDNS
- Подводные камни: **DNS propagation** (TTL кэш, клиент может ходить на старый IP часами), **search domains** в resolv.conf (неожиданные резолвы), **/etc/hosts** приоритетнее DNS

---

## Go: два резолвера

```go
// Pure Go (дефолт на Linux)
// + кроссплатформенный
// + не требует CGO
// + горутины, не блокирует M
// - не поддерживает mDNS, сложный nsswitch.conf

// CGO resolver
// + полная совместимость с libc (mDNS, nsswitch plugins)
// - блокирует M (CGO = syscall)
// - нужен CGO_ENABLED=1

// Принудительно выбрать:
GODEBUG=netdns=go    // pure Go
GODEBUG=netdns=cgo   // CGO
```

```go
// Кастомный резолвер
resolver := &net.Resolver{
    PreferGo: true,  // pure Go
    Dial: func(ctx context.Context, network, address string) (net.Conn, error) {
        d := net.Dialer{Timeout: 5 * time.Second}
        return d.DialContext(ctx, "udp", "8.8.8.8:53")  // свой DNS-сервер
    },
}

ips, err := resolver.LookupHost(ctx, "example.com")
```

## DNS-based Load Balancing

```
example.com → A 1.2.3.4
example.com → A 5.6.7.8
example.com → A 9.10.11.12
```

DNS отдаёт записи в **рандомном** порядке (round-robin). Клиент обычно берёт первый IP.

**Проблемы:**
- Клиент **кэширует** и ходит на один IP → неравномерная нагрузка
- TTL не гарантирует: Java кэширует DNS **навсегда** по дефолту
- Нет health check: DNS не знает что сервер упал
- Удалил A-запись → клиенты с кэшем **всё ещё ходят** на старый IP

**Go-специфика**: `net.Dialer` кэширует DNS на ~5 минут. HTTP client переиспользует соединения (keep-alive) → DNS resolve только при новом connect.

## Service Discovery

**SRV-записи**:
```
_grpc._tcp.myservice.example.com → 10 50 8080 server1.example.com
                                    │   │   │    │
                                 priority weight port host
```

```go
_, addrs, _ := net.LookupSRV("grpc", "tcp", "myservice.example.com")
for _, a := range addrs {
    fmt.Printf("%s:%d (priority=%d, weight=%d)\n", a.Target, a.Port, a.Priority, a.Weight)
}
```

**Consul DNS**: `myservice.service.consul` → IP:port. Встроенный health check.

**Kubernetes CoreDNS**: `myservice.mynamespace.svc.cluster.local` → ClusterIP.

## Подводные камни

### search domains (resolv.conf)

```
# /etc/resolv.conf
nameserver 8.8.8.8
search mycompany.com dev.mycompany.com

# dig api → попробует: api.mycompany.com, api.dev.mycompany.com, api
```

В Kubernetes: `search default.svc.cluster.local svc.cluster.local cluster.local`. Запрос `api` пробует 4 варианта → **лишние DNS-запросы**.

Фикс в K8s: `ndots: 1` или FQDN с точкой: `api.external.com.` (trailing dot = абсолютное имя).

### DNS Propagation

Изменил A-запись → часть клиентов **часами** ходит на старый IP (кэш recursive resolver'а, кэш ОС, кэш приложения).

**Стратегия миграции**: снизить TTL **заранее** (за 1 день до миграции TTL=60s) → мигрировать → вернуть TTL.

### DNS Rebinding

Атака: DNS возвращает внутренний IP (127.0.0.1, 10.x.x.x) → браузер обращается к внутреннему сервису от имени внешнего домена.

## Связь
- [[DNS — как работает]] — протокол, иерархия, типы записей
- [[Что происходит при curl https]] — DNS = шаг 2
- [[Syscalls]] — CGO resolver блокирует M
