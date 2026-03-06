#flashcards/rwmutex/example

В начале: `readerCount=0`, `readerWait=0`. R1, R2, R3 вызывают `RLock()`. Какими станут все поля?
?
![[пример#^example-readers-enter]]

R1, R2, R3 внутри (readerCount=3). W1 вызывает `Lock()`. Опиши все 4 шага и финальные значения полей.
?
![[пример#^example-writer-waits]]

W1 ждёт (readerCount=-999_999_997). R4 вызывает `RLock()`. Что произойдёт и почему?
?
![[пример#^example-r4-blocks]]

R1 вызывает `RUnlock()` (readerWait=3). Затем R2. Почему они не будят писателя?
?
![[пример#^example-r1r2-leave]]

R3 — последний. Он вызывает `RUnlock()`. Почему именно он будит W1?
?
![[пример#^example-r3-last]]

W1 заканчивает и вызывает `Unlock()`. `readerCount = -999_999_999`. Опиши три шага `Unlock()` и почему R4 просыпается.
?
![[пример#^example-writer-unlock]]

Итоговый вопрос: R4 был в очереди всё время пока работал W1. Это честно? Мог ли другой писатель W2 вклиниться между W1 и R4?
?
Нет, не мог. `Unlock()` сначала делает `Semrelease(readerSem)` для всех ждущих читателей, и только потом `w.Unlock()`. Следующий писатель W2 ждёт на `rw.w.Lock()` — он получит доступ только после `w.Unlock()`, к тому моменту R4 уже проснулся и может работать параллельно.
![[пример#^example-writer-unlock]]
![[Unlock()#^unlock-reader-starvation]]
