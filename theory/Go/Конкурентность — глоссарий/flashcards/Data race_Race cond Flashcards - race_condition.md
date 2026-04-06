#flashcards/dr_and_rc/race_condition

Что такое race condition? Назови необходимые условия.
?
![[Race Condition#^rc-definition]]

Назови 3 типичных паттерна race condition и способы их исправления.
?
![[Race Condition#^rc-fixes]]

Почему race condition может существовать без data race? Приведи пример.
?
![[Race Condition#^rc-no-dr]]

Что выведет этот код (конкурентно вызывается 2 горутинами)?
```go
func (s *Server) increment() {
    x := s.get()   // читаем: 5
    s.set(x + 1)
}
// s.get/set защищены mutex внутри
```
?
Ожидаем итог 7 (5+1+1), получаем 6. Race condition без data race — обе горутины прочитали 5, обе записали 6. Mutex внутри get/set не помогает — проблема в составной операции.
![[Race Condition#^rc-no-dr-example]]

Что такое паттерн check-then-act и почему он опасен?
?
![[Race Condition#^rc-check-then-act]]

Почему в примере с transfer два mutex (по одному на каждый аккаунт) не спасают от race condition?
?
![[Race Condition#^rc-transfer-example]]

Что выведет этот код?
```go
type SafeCounter struct {
    mu sync.Mutex
    v  int
}

func (c *SafeCounter) Inc() {
    c.mu.Lock()
    c.v++
    c.mu.Unlock()
}

func (c *SafeCounter) IncIfLess(limit int) {
    if c.v < limit {   // проверка без lock!
        c.Inc()
    }
}
```
?
Это race condition (и data race на `c.v` в `IncIfLess`). Проверка `c.v < limit` идёт без lock — другая горутина может изменить `c.v` между проверкой и `Inc()`. Нужно взять lock на всю операцию.
![[Race Condition#^rc-check-then-act]]
