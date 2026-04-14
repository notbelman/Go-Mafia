#flashcards/rwmutex/lock

Как `Lock()` блокирует новых читателей не трогая `readerSem`?
?
![[sync.RWMutex/Lock()#^lock-block-readers]]

После `rw.readerCount.Add(-rwmutexMaxReaders)` результат отрицательный. Как из него получить реальное число активных читателей?
?
![[sync.RWMutex/Lock()#^lock-r-formula]]

Почему в `Lock()` используется именно `rwmutexMaxReaders ≈ 1_000_000_000`? Почему не `1` или `math.MaxInt32`?
?
Значение должно быть достаточно большим чтобы гарантированно сделать `readerCount` отрицательным при любом количестве реальных читателей (до ~1 млрд), но при этом оставлять нижние биты для хранения реального числа ожидающих читателей. `math.MaxInt32` не подойдёт — при прибавлении реальных читателей переполнится знаковый бит.
![[sync.RWMutex/Lock()#^lock-r-formula]]

`Lock()` вызван, `r = 3`. Пока выполняется следующая строка, все 3 читателя успевают вызвать `RUnlock()`. Что произойдёт — писатель заблокируется?
?
![[sync.RWMutex/Lock()#^lock-two-conditions]]
![[sync.RWMutex/Lock()#^lock-two-conditions-detail]]

Зачем в `Lock()` два условия (`r != 0 && readerWait.Add(r) != 0`)? Что проверяет каждое?
?
![[sync.RWMutex/Lock()#^lock-two-conditions-detail]]

Какую роль играет внутренний `rw.w.Lock()` в начале `Lock()`? Что будет если два писателя вызовут `Lock()` одновременно?
?
`rw.w` — обычный `sync.Mutex`. Он сериализует писателей между собой. Второй писатель заблокируется на `rw.w.Lock()` и не дойдёт до манипуляций с `readerCount`. Таким образом `readerCount` вычитает `rwmutexMaxReaders` только один раз — для текущего активного писателя.
![[sync.RWMutex/Lock()#^lock-block-readers]]
