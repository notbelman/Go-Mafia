#flashcards/memory/reordering

Почему reordering не нарушает результат в рамках одного потока, но ломает логику другого?
?
![[Reordering инструкций#^reorder-single-thread]]
+
![[Reordering инструкций#^reorder-rule]]

Что такое out-of-order execution и зачем процессор это делает?
?
![[Reordering инструкций#^reorder-cpu-ooo]]

Что такое store buffer и почему он создаёт проблему видимости между ядрами?
?
![[Reordering инструкций#^reorder-store-buffer]]

Почему компилятор реордерит инструкции и почему это создаёт проблему при многопоточности?
?
![[Reordering инструкций#^reorder-compiler]]

Что произойдёт в этом коде — будет ли msg всегда "hello"? Почему?
```go
var msg string
var flag bool
go func() { msg = "hello"; flag = true }()
for !flag { runtime.Gosched() }
fmt.Println(msg)
```
?
![[Reordering инструкций#^reorder-example-goroutine1]]
![[Reordering инструкций#^reorder-example-goroutine2]]

Что означает data race с точки зрения reordering? Почему примитивы синхронизации решают эту проблему?
?
![[Reordering инструкций#^reorder-sync-primitives]]

Почему баги от reordering называют "термоядерными"? Назови все 4 причины почему они не воспроизводятся стабильно.
?
![[Reordering инструкций#^reorder-arch-diff]]
![[Reordering инструкций#^reorder-compiler-version]]
![[Reordering инструкций#^reorder-timing]]
![[Reordering инструкций#^reorder-repro-rate]]

С какой частотой воспроизводятся баги от reordering в типичном случае?
?
![[Reordering инструкций#^reorder-repro-rate]]
