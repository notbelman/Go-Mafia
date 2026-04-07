#flashcards/sync-patterns/cas

Что такое CAS (Compare-And-Swap)? Опиши механизм атомарно.
?
![[CAS_паттерны#^cas-hw-impl]]

На каких инструкциях реализован CAS на x86 и ARM?
?
![[CAS_паттерны#^cas-hw-impl]]

Почему CAS почти всегда используется в цикле? Что произойдёт без цикла?
?
![[CAS_паттерны#^cas-loop-why]]

Что такое ABA проблема? Опиши сценарий по шагам.
?
![[CAS_паттерны#^aba-problem]]

Как версионный счётчик решает ABA проблему?
?
![[CAS_паттерны#^aba-solution-version]]

Какие два других подхода решают ABA кроме версионного счётчика? Для чего они применяются?
?
![[CAS_паттерны#^aba-solution-hazard]]

Как решается ABA проблема в Go?
?
![[CAS_паттерны#^aba-solution-go]]

Опиши паттерн "lock-free флаг one-shot" на CAS. Почему именно CompareAndSwap, а не Store?
?
![[CAS_паттерны#^cas-pattern-oneshot]]

Опиши паттерн copy-on-write с CAS. Зачем нужен цикл в UpdateConfig?
?
![[CAS_паттерны#^cas-pattern-cow]]

Как выглядит spin lock реализованный через CAS?
?
![[CAS_паттерны#^cas-pattern-spinlock]]

Как реализован lock-free push для стека через CAS? Почему node.next нужно устанавливать до CAS?
?
![[CAS_паттерны#^cas-pattern-stack]]

Чем CAS лучше мьютекса при низком contention? Чем хуже при высоком?
?
![[CAS_паттерны#^cas-vs-mutex]]

Почему CAS loop деградирует при высоком contention? Что происходит на уровне кэша?
?
![[CAS_паттерны#^cas-high-contention]]

Что выведет этот код?
```go
var v atomic.Int64

var wg sync.WaitGroup
for i := 0; i < 3; i++ {
    wg.Add(1)
    go func() {
        defer wg.Done()
        for {
            old := v.Load()
            if v.CompareAndSwap(old, old+1) {
                return
            }
        }
    }()
}
wg.Wait()
fmt.Println(v.Load())
```
?
`3` — каждая горутина делает CAS loop до победы, финальное значение = 3. CAS гарантирует что каждый инкремент выполнится ровно один раз несмотря на гонку.
![[CAS_паттерны#^cas-loop-why]]
