#flashcards/lock-free-and-sync/msqueue

Что такое очередь Майкла-Скотта? Почему у неё меньше contention чем у стека Трайбера?
?
![[Очередь Майкла-Скотта#^msq-def]] + ![[Очередь Майкла-Скотта#^msq-less-contention]]

Сколько CAS нужно для Push? Что они обновляют?
?
![[Очередь Майкла-Скотта#^msq-push-two-cas]]

Что делает горутина если видит tail.next ≠ nil при начале Push?
?
![[Очередь Майкла-Скотта#^msq-help-mechanism]]

Зачем нужен dummy-элемент при инициализации очереди?
?
![[Очередь Майкла-Скотта#^msq-dummy]]

Почему успех CAS 2 (обновление tail) не проверяется в Push?
?
![[Очередь Майкла-Скотта#^msq-cas2-unchecked]]

Pop видит head == tail, но next ≠ nil. Что это означает и что делает Pop?
?
![[Очередь Майкла-Скотта#^msq-pop-helps]]

Сформулируй общий паттерн для lock-free операций с >1 CAS.
?
![[Очередь Майкла-Скотта#^msq-pattern]]

Что выведет этот код?
```go
q := NewQueue()
q.Push(1)
q.Push(2)
q.Push(3)

fmt.Println(q.Pop())
fmt.Println(q.Pop())
q.Push(4)
fmt.Println(q.Pop())
fmt.Println(q.Pop())
fmt.Println(q.Pop())
```
?
`1 2 3 4 -1` — очередь FIFO. Pop возвращает -1 на пустой очереди. Head (pop) и tail (push) — разные указатели, поэтому Push и Pop почти не конкурируют.
![[Очередь Майкла-Скотта#^msq-less-contention]]
