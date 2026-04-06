- набор метаданных, ассоциированных с запросом или процессом
- позволяет: отменять работу горутин, ограничивать время выполнения, пробрасывать метаданные (trace ID, user ID)
- природа матрёшки: отмена родителя → отмена всех детей; отмена ребёнка НЕ влияет на родителя и соседей

---
[[Context Flashcards - context]]
## Интерфейс

```go
type Context interface {
    Done() <-chan struct{}        // закрывается при отмене
    Err() error                   // nil → Canceled | DeadlineExceeded
    Deadline() (time.Time, bool)  // дедлайн, если установлен
    Value(key any) any            // значение по ключу (поиск вверх по дереву)
}
```

Любой тип, реализующий эти 4 метода, является контекстом. Можно написать свой. ^ctx-interface-def

## Дерево контекстов

```
Background (корень, никогда не отменяется)
    └── WithCancel (request)
            ├── WithTimeout (db query, 2s)
            │       └── WithValue (trace_id)
            └── WithTimeout (http call, 5s)
```

Отменяем request → отменяются db query и http call по цепочке. ^ctx-tree-cancel-down

Отменяем db query → request и http call живут дальше. ^ctx-tree-cancel-isolated

## Назначение context

Context — набор метаданных, ассоциированных с запросом или процессом. Позволяет отменять работу горутин, ограничивать время выполнения и пробрасывать метаданные (trace ID, user ID). ^ctx-purpose

## Природа матрёшки

Отмена родителя → отмена всех детей. Отмена ребёнка НЕ влияет на родителя и соседей. ^ctx-propagation

## Связь
- [[context WithCancel]] — ручная отмена
- [[context WithTimeout]] — автоотмена по времени
- [[context WithValue]] — передача метаданных
- [[Context ошибки и правила]] — антипаттерны
