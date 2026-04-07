#flashcards/sync-patterns/recursive_timed_mutex

Что такое recursive mutex? Чем он отличается от обычного?
?
![[Recursive_и_Timed_mutex#^recursive-def]]

Что произойдёт если владелец обычного sync.Mutex вызовет Lock() повторно в Go?
?
Дедлок. Обычный мьютекс не рекурсивный — повторный Lock() заблокирует поток навсегда. Это намеренное решение в Go.
![[Recursive_и_Timed_mutex#^recursive-def]]

Почему в Go нет публичного goroutine ID и как это связано с recursive mutex?
?
![[Recursive_и_Timed_mutex#^recursive-no-goid]]

Назови три причины почему Go против recursive mutex.
?
![[Recursive_и_Timed_mutex#^recursive-bad-design]]
![[Recursive_и_Timed_mutex#^recursive-unlock-errors]]
![[Recursive_и_Timed_mutex#^recursive-go-alternative]]

Какой Go-идиоматичный паттерн заменяет recursive mutex?
?
![[Recursive_и_Timed_mutex#^recursive-go-alternative]]

Что такое timed mutex? Зачем он нужен?
?
![[Recursive_и_Timed_mutex#^timed-why-deadlock]]
![[Recursive_и_Timed_mutex#^timed-why-graceful]]
![[Recursive_и_Timed_mutex#^timed-why-sla]]

Как реализовать timed mutex на Go через каналы?
?
![[Recursive_и_Timed_mutex#^timed-impl]]

Какой Go-идиоматичный подход заменяет timed mutex?
?
![[Recursive_и_Timed_mutex#^timed-go-idiomatic]]

Почему в Go нет timed mutex в stdlib?
?
![[Recursive_и_Timed_mutex#^go-no-timed]]

Что выведет этот код?
```go
var mu sync.Mutex

func foo() {
    mu.Lock()
    defer mu.Unlock()
    bar()
}

func bar() {
    mu.Lock()
    defer mu.Unlock()
    fmt.Println("in bar")
}

foo()
```
?
Дедлок (fatal error: all goroutines are asleep). `bar()` пытается взять уже захваченный мьютекс — sync.Mutex не рекурсивный. Программа зависнет навсегда.
![[Recursive_и_Timed_mutex#^recursive-def]]
