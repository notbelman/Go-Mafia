#flashcards/GMP/preemption

Какую модель вытеснения использует Go? Из каких двух частей она состоит?
?
![[preemption#Кооперативная часть]]
+
![[preemption#Асинхронно-вытесняющая часть (Go 1.14+)]]

Перечисли все кооперативные точки переключения в Go
?
![[preemption#^coop-yield-points]]

Кто вставляет кооперативные точки в Go-код и почему программист не делает это вручную?
?
![[preemption#^coop-compiler-inserts]]

Опиши пошагово механизм асинхронного вытеснения в Go 1.14+
?
![[preemption#^fa7cbc]]
![[preemption#^async-preempt-mechanism]]

С какой версии Go появилось асинхронное вытеснение и через какой сигнал оно работает?
?
![[preemption#Асинхронно-вытесняющая часть (Go 1.14+)]]

Почему вытеснение называется «асинхронным», если оно происходит через сигнал?
?
![[preemption#^async-preempt-delay]]

Что такое safe point? Почему нельзя остановить горутину в произвольный момент?
?
![[preemption#^safe-points]]

Что произойдёт с этим кодом в Go 1.13 vs Go 1.14+?
```go
go func() {
    for { x++ }
}()
time.Sleep(time.Second)
fmt.Println("done")
```
?
Go 1.13: `fmt.Println` может никогда не выполниться — горутина захватит P навсегда, нет кооперативных точек в tight loop. Go 1.14+: sysmon вытеснит горутину через ~10ms, `fmt.Println` выполнится.
![[preemption#^preempt-before-1-14]]
![[preemption#^preempt-after-1-14]]

Что делает `runtime.Gosched()`? Чем отличается от `time.Sleep(0)`?
?
![[preemption#^gosched]]

В чём принципиальный риск кооперативного планировщика (без вытеснения)?
?
![[preemption#^preempt-before-1-14]]

Какой overhead у вытесняющего планировщика по сравнению с кооперативным и почему?
?
![[preemption#Кооперативная часть]]
