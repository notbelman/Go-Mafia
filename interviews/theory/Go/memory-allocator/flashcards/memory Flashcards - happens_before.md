#flashcards/memory/happens_before

Что означает "A happens-before B"?
?
![[Happens-before через atomic#^hb-definition]]

Какую happens-before связь создаёт atomic.Store в Go?
?
![[Happens-before через atomic#^hb-atomic-store-load]]

Что позволяет синхронизировать atomic флаг помимо самого флага?
?
![[Happens-before через atomic#^hb-sync-nonatomic]]

Что не так с этим кодом? Что именно может пойти не так и почему?
```go
var msg string
var flag bool
go func() { msg = "hello"; flag = true }()
for !flag { runtime.Gosched() }
fmt.Println(msg)
```
?
![[Happens-before через atomic#^hb-problem-example]]

Нарисуй цепочку happens-before для atomic-решения с флагом. Почему (D) гарантированно видит запись (A)?
?
![[Happens-before через atomic#^9ba76b]]
![[Happens-before через atomic#^hb-solution-chain]]

Почему race detector не ругается на код с atomic флагом, хотя msg — обычная переменная?
?
![[Happens-before через atomic#^hb-race-detector]]

Чем Go atomic отличается от C++ atomic по выбору memory order? Какой конкретно memory order используется в Go?
?
![[Happens-before через atomic#^hb-go-full-barrier]]

Какой трейдофф у подхода Go "все atomic = полный барьер" по сравнению с C++?
?
![[Happens-before через atomic#^hb-go-vs-cpp]]

Какую happens-before гарантию даёт отправка в канал (`ch <- v`) и получение (`<-ch`)?
?
![[Happens-before через atomic#^hb-channel]]

Какую happens-before гарантию даёт мьютекс?
?
![[Happens-before через atomic#^hb-mutex]]

Какую happens-before гарантию даёт запуск горутины (`go f()`)?
?
![[Happens-before через atomic#^hb-goroutine-start]]

Какую happens-before гарантию даёт `sync.Once.Do(f)`?
?
![[Happens-before через atomic#^hb-once]]

В каких трёх случаях оправдано использовать синхронизацию через atomic ordering вместо мьютекса?
?
![[Happens-before через atomic#^hb-use-hot-path]]
![[Happens-before через atomic#^hb-use-flag]]
![[Happens-before через atomic#^hb-use-understand]]

Почему в большинстве случаев мьютекс предпочтительнее atomic ordering?
?
![[Happens-before через atomic#^hb-default-mutex]]
