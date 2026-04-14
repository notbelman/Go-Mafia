#flashcards/channels-patterns/promise_future

Что такое Promise в JS-стиле? Как устроен Then()?
?
![[Promise и Future#^promise-js-def]]

Как сделать Then() неблокирующим?
?
![[Promise и Future#^promise-then-nonblock]]

Что такое Future в C++-стиле? Как работает Get()?
?
![[Promise и Future#^future-def]]

В чём разница ролей Promise и Future в паттерне Promise+Future?
?
![[Promise и Future#^promise-future-roles]]

Опиши аналогию для Promise+Future.
?
![[Promise и Future#^promise-future-analogy]]

Зачем Promise/Future если есть каналы?
?
![[Promise и Future#^promise-why]]

Что выведет этот код?
```go
f := NewFuture(func() interface{} {
    time.Sleep(100 * time.Millisecond)
    return 42
})
fmt.Println("before get")
fmt.Println(f.Get())
fmt.Println("after get")
```
?
```
before get
42
after get
```
`before get` — немедленно, так как NewFuture запускает горутину и сразу возвращает. Затем `f.Get()` блокируется на 100ms пока задача не завершится, потом печатает 42.
![[Promise и Future#^future-def]]
