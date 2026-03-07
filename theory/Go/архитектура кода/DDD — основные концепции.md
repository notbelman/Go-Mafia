## Суть

Код моделирует бизнес-домен. Термины в коде = термины из разговоров с бизнесом.

## Проблема

Разработчик пишет `CreateUserHandler`, `UpdateOrderService`, `ProcessPayment` — технические имена. Бизнес говорит "оформить заказ", "отменить бронь". Код не отражает домен → разработчик и бизнес говорят на разных языках → баги, недопонимание.

## Два уровня

**Strategic — как делить систему:**

- **Ubiquitous Language** — единый словарь. `Order`, `Shipment`, `refund()` — бизнес-понятия, не технические абстракции
- **Bounded Context** — граница, внутри которой язык консистентен. Order в Checkout = корзина + оплата. Order в Shipping = адрес + трекинг. Это разные модели.

**Tactical — как писать код:**

- **Entity** — уникальный ID, жизненный цикл, мутабельная. `User`, `Order`
- **Value Object** — без ID, сравнивается по полям, иммутабельный. `Money`, `Address`, `Email`
- **Aggregate** — кластер entity + value objects, граница транзакции (отдельная карточка)
- **Domain Service** — логика между entities: `TransferMoney(from, to)`

## Go-пример

```go
// Entity — сравниваем по ID
type Order struct { id string; status Status; items []Item }

// Value Object — сравниваем по значению, иммутабельный
type Money struct { Amount int; Currency string }
func (m Money) Add(other Money) Money { return Money{m.Amount + other.Amount, m.Currency} }
```

## Когда оверкилл

CRUD без бизнес-логики, маленькие проекты. DDD окупается когда есть нетривиальные бизнес-правила.
