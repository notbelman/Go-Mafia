Две стадии: в первой собираем бинарник, во вторую копируем только его.
```dockerfile
# --- Стадия 1: сборка ---
FROM golang:1.22-alpine AS builder

WORKDIR /app

# Сначала зависимости (кэшируются отдельно от кода)
COPY go.mod go.sum ./
RUN go mod download

# Потом код (меняется чаще)
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o /app/server ./cmd/server

# --- Стадия 2: финальный образ ---
FROM alpine:3.19

COPY --from=builder /app/server /server

ENTRYPOINT ["/server"]
```

**Почему зависимости отдельно от кода:**
```
COPY go.mod go.sum → go mod download → COPY . . → go build

Изменил код, но не зависимости:
  go mod download — берётся из кэша (слой не изменился)
  go build — пересобирается

Если COPY . . сделать до go mod download:
  любое изменение кода инвалидирует кэш зависимостей
  каждая сборка качает всё заново
```

**Размер образа:**
```
golang:1.22       ~800 MB (SDK, тулчейн, всё)
alpine:3.19       ~7 MB + бинарник
scratch           ~0 MB + бинарник (нет shell, нет отладки)
```

`CGO_ENABLED=0` — статическая линковка, бинарник не зависит от libc. Можно запускать в scratch.