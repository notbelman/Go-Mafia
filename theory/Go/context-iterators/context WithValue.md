- передача request-scoped метаданных через границы API: trace ID, user ID, auth token ^wv-usecase
- поиск по ключу идёт вверх по дереву (связанный список) — O(n) по глубине ^wv-lookup-complexity
- ребёнок может перекрыть ключ родителя (shadowing) → баги через месяцы/годы ^wv-shadowing-danger

---

## Базовый паттерн

```go
type traceIDKey struct{}  // неэкспортируемый тип!

// записать
ctx = context.WithValue(parent, traceIDKey{}, "abc-123")

// прочитать (обёртка с type assertion)
func TraceID(ctx context.Context) (string, bool) {
    val, ok := ctx.Value(traceIDKey{}).(string)
    return val, ok
}
```

Неэкспортируемый тип ключа гарантирует, что никто из другого пакета не сможет случайно перекрыть значение. ^wv-unexported-key-why

## Что класть, а что нет

Контекст = метаданные запроса. Не бизнес-логика, не зависимости. ^wv-what-to-store

| Да (метаданные) | Нет (явные аргументы) |
|:----------------|:----------------------|
| trace ID, request ID | конфиг приложения |
| user ID, auth token | DB connection, logger |
| идентификатор транзакции | параметры функции |

Overhead: обход связанного списка + type assertion из пустого интерфейса. Если логер задан в корне дерева, от дочернего контекста придётся пройти всю цепочку. ^wv-overhead

## Коллизия ключей — опасная бага

```go
// Пакет A:
ctx = context.WithValue(ctx, "userID", 123)

// Пакет B (или библиотека):
ctx = context.WithValue(ctx, "userID", "admin")  // перекрыл!

ctx.Value("userID")  // "admin" — пакет A потерял своё значение
```

Баг проявляется не сразу — через месяцы, когда кто-то подключит библиотеку с таким же строковым ключом. ^wv-collision-timing

## Решение: type definition для ключа

```go
// Пакет A:
type userIDKeyA struct{}
ctx = context.WithValue(ctx, userIDKeyA{}, 123)

// Пакет B:
type userIDKeyB struct{}
ctx = context.WithValue(ctx, userIDKeyB{}, "admin")

// Нет коллизии! Сравнение интерфейсов = тип + значение
ctx.Value(userIDKeyA{})  // 123 — на месте
ctx.Value(userIDKeyB{})  // "admin" — тоже на месте
```

Сравнение ключей в context.Value работает через сравнение интерфейсов: тип + значение. Два разных структурных типа никогда не будут равны, даже если оба `struct{}`. ^wv-interface-comparison

## Поиск вверх по дереву

```go
ctx1 := context.WithValue(bg, traceKey{}, "abc")
ctx2 := context.WithoutCancel(ctx1)  // нет своих values

ctx2.Value(traceKey{})  // "abc" — нашёл у родителя
```

Если ключ не найден ни у кого в цепочке → Value() вернёт nil (не паника). ^wv-nil-not-panic

## Связь
- [[Context]] — дерево контекстов
- [[context WithoutCancel]] — сохраняет цепочку Value, обрывает отмену
- [[Context ошибки и правила]] — WithValue только для request-scoped данных
