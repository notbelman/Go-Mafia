## Главная мысль

Все три решают проблему Layered Architecture: **переворачивают зависимости**. Бизнес-логика в центре, инфраструктура (БД, HTTP, очереди) снаружи. Зависимости направлены **только внутрь**.

## Три имени — одна суть

||Hexagonal (2005)|Onion (2008)|Clean (2012)|
|:--|:--|:--|:--|
|Автор|Alistair Cockburn|Jeffrey Palermo|Robert Martin|
|Центр|Application core|Domain Model|Entities|
|Интерфейсы|Ports|Domain Services interfaces|Use Cases boundaries|
|Реализации|Adapters|Infrastructure|Frameworks & Drivers|
|Фокус|Порты и адаптеры|Слои вокруг домена|Dependency Rule между слоями|

**Общее**: бизнес-логика не зависит от БД/фреймворков, зависимости только внутрь, DIP на уровне архитектуры.

## Port и Adapter — ключевой механизм

**Port** — интерфейс, который определяет бизнес-логика (что ей нужно от внешнего мира). **Adapter** — реализация этого интерфейса конкретной технологией.

```
// Port (определяет domain)
type UserRepository interface {
    FindByID(id string) (*User, error)
}

// Adapter (реализует infrastructure)
type PostgresUserRepo struct { db *sql.DB }
func (r *PostgresUserRepo) FindByID(id string) (*User, error) { ... }
```

Смена БД = новый адаптер. Бизнес-логика не меняется.

## Зачем — vs Layered

В Layered: business → data access (бизнес зависит от БД). Здесь: business определяет интерфейс ← data access реализует его (БД зависит от бизнеса).