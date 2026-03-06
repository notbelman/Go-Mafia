## Таблица защиты

| Уровень | Dirty Read | Lost Update | Non-Repeatable | Phantom |
|---------|------------|-------------|----------------|---------|
| READ COMMITTED (дефолт) | ✓ | - | - | - |
| REPEATABLE READ | ✓ | ✓ | ✓ | ✓* |
| SERIALIZABLE | ✓ | ✓ | ✓ | ✓ |

*PostgreSQL строже стандарта SQL - фантомы закрыты на REPEATABLE READ благодаря MVCC

## Как работает в PostgreSQL

| Уровень | Механизм |
|---------|----------|
| READ COMMITTED | Новый snapshot на каждый SELECT |
| REPEATABLE READ | Один snapshot на всю транзакцию (фиксируется на первом запросе) |
| SERIALIZABLE | Snapshot + predicate locks для отслеживания зависимостей |

## Производительность

Выше изоляция → больше накладных расходов → больше конфликтов

| Уровень | Цена |
|---------|------|
| READ COMMITTED | Минимальная, нет serialization failures |
| REPEATABLE READ | Snapshot живёт дольше, блокирует VACUUM, возможны failures |
| SERIALIZABLE | Predicate locks + память, больше retry |

**Главное правило:** не держи транзакции открытыми долго.