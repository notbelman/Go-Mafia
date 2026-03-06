- **Race condition** = корректность зависит от тайминга. **Логическая** ошибка, даже если память синхронизирована ^rc-def
- Может быть **без data race**: все обращения через mutex/channel, но порядок операций ломает результат ^rc-without-dr
- `-race` **не ловит** race condition — это не про память, а про логику ^rc-no-detector

---

## Классика: check-then-act

```go
// ПЛОХО — race condition (даже с мьютексами!)
func transfer(from, to *Account, amount int) {
    from.mu.Lock()
    ok := from.balance >= amount
    from.mu.Unlock()
    // ← ТУТ другая горутина может обнулить from.balance
    if ok {
        from.mu.Lock()
        from.balance -= amount
        from.mu.Unlock()
    }
}
```
^rc-check-then-act

**Проблема**: между проверкой и действием состояние может измениться. ^rc-gap

```go
// ХОРОШО — один лок на всю операцию
func transfer(from, to *Account, amount int) bool {
    mu.Lock()
    defer mu.Unlock()
    if from.balance < amount { return false }
    from.balance -= amount
    to.balance += amount
    return true
}
```
^rc-fix

## Может быть БЕЗ data race

```go
func (s *Server) increment() {
    x := s.get()   // через channel, data race нет
    s.set(x + 1)   // через channel, data race нет
}
// Две горутины: обе прочитали 5, обе записали 6. Ожидали 7.
```
^rc-no-dr-example

## Как исправить

| Проблема | Решение |
|:--|:--|
| check-then-act | один лок на всю логику |
| read-modify-write | `atomic.AddInt64` или mutex на всю операцию |
| несколько ресурсов | один mutex на все или транзакция |
^rc-fixes

## Связь
- [[Data Race]] — memory error, не logic error
- [[Critical Section]] — КС должна покрывать всю логику, не кусочки
