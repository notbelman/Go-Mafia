#flashcards/sync-primitive-supple/livelock_starvation

Что такое livelock? Чем он отличается от deadlock?
?
![[Livelock_и_Starvation#^livelock-definition]]

Что такое starvation? Чем он отличается от livelock?
?
![[Livelock_и_Starvation#^starvation-definition]]

Сравни deadlock / livelock / starvation по трём параметрам: состояние горутин, ресурсы, прогресс.
?
![[Livelock_и_Starvation#^comparison-table]]

В коде ниже горутины вызывают Gosched() и продолжают цикл. Это deadlock, livelock или starvation? Почему?
```go
mu1.Lock()
if !mu2.TryLock() {
    mu1.Unlock()
    runtime.Gosched()
    continue
}
```
?
Livelock — горутины активны, не заблокированы, постоянно берут и отдают ресурсы, но полезной работы = 0, прогресса нет.
![[Livelock_и_Starvation#^livelock-code]]

В каких ситуациях чаще всего встречается livelock в Go?
?
![[Livelock_и_Starvation#^livelock-where]]

Как возникает starvation на примере жадного и вежливого воркеров? Объясни механизм.
?
![[Livelock_и_Starvation#^starvation-example]]

Только ли с памятью бывает starvation? Какие ещё ресурсы могут голодать?
?
![[Livelock_и_Starvation#^starvation-resources]]

Почему TryLock в цикле может привести к livelock? Как это связано с Gosched?
?
![[Livelock_и_Starvation#^livelock-code]]

Что такое starvation mode мьютекса (Go 1.9)? Какую проблему он решает?
?
![[Livelock_и_Starvation#^three-problems-comparison]]
