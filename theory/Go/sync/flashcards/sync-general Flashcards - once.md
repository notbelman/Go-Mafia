#flashcards/sync-general/once

Что гарантирует sync.Once?
?
![[sync.Once#^once-definition]]

Опиши fast path и slow path в sync.Once.Do().
?
![[sync.Once#^once-fast-path]]
![[sync.Once#^once-slow-path]]

Почему sync.Once использует mutex, а не CAS для блокировки? В чём проблема с CAS?
?
![[sync.Once#^once-why-mutex-not-cas]]

Что произойдёт если внутри функции f() вызвать once.Do() рекурсивно?
?
![[sync.Once#^once-recursive-deadlock]]

Почему поле `done` стоит первым в структуре Once?
?
![[sync.Once#^once-done-first-field]]

Опиши внутреннюю структуру sync.Once.
?
![[sync.Once#^once-struct]]

Опиши алгоритм Do() по шагам — fast path и slow path с double-check.
?
![[sync.Once#^once-algorithm]]

Что выведет этот код?
```go
var once sync.Once
var result int

for i := 0; i < 5; i++ {
    i := i
    go func() {
        once.Do(func() {
            result = i
        })
    }()
}
time.Sleep(time.Millisecond)
fmt.Println(result)
```
?
Выведет значение `i` той горутины, которая первой выполнила `Do()` — это может быть любое из 0-4. Все остальные горутины увидят `done == true` и пропустят. Результат недетерминирован, но `Do()` выполнится ровно один раз.
![[sync.Once#^once-definition]]
