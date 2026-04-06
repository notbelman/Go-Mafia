#flashcards/dr_and_rc/happens_before

Что означает "A happens-before B"?
?
![[Happens-Before в Go#^hb-definition]]

Перечисли все happens-before гарантии в Go (6 штук).
?
![[Happens-Before в Go#^hb-guarantees]]

Что гарантирует `close(ch)` с точки зрения happens-before?
?
![[Happens-Before в Go#^hb-guarantees]]

Что гарантирует `once.Do(f)` с точки зрения happens-before?
?
![[Happens-Before в Go#^hb-guarantees]]

Гарантирует ли `atomic.Store` happens-before для `atomic.Load`?
?
![[Happens-Before в Go#^hb-guarantees]]

Объясни цепочку happens-before в этом примере: запись `a = "hello"` → send → receive → `print(a)`.
?
![[Happens-Before в Go#^hb-chain]]

Что выведет этот код?
```go
var a string
var done = make(chan bool)

go func() {
    a = "hello"
    done <- true
}()

<-done
print(a)
```
?
`hello`. Send happens-before receive (channel guarantee), запись `a` sequenced-before send → значит `a = "hello"` happens-before `print(a)`.
![[Happens-Before в Go#^hb-example]]

Что выведет этот код?
```go
var a string
var done bool

go func() {
    a = "hello"
    done = true
}()

for !done {}
print(a)
```
?
Результат непредсказуем (UB / data race). Нет happens-before между записью `done = true` и чтением `done` в main. Компилятор или CPU могут переупорядочить или кэшировать. Нужен channel или atomic.
![[Happens-Before в Go#^hb-definition]]
