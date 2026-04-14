#flashcards/mutex/unlock

Что делает fast path в Unlock()? При каком условии он завершается без slow path?
?
![[sync.Mutex/Unlock()#^unlock-fast-path]]

В Normal mode Unlock() иногда не будит ни одну горутину — при каких четырёх условиях?
?
![[sync.Mutex/Unlock()#^unlock-slow-normal-skip]]

Что делает Unlock() в Normal mode когда решает разбудить горутину? Какой handoff-флаг передаётся?
?
![[sync.Mutex/Unlock()#^unlock-slow-normal-wake]]

Чем отличается Unlock() в Starvation mode от Normal? Что означает handoff=true?
?
![[sync.Mutex/Unlock()#^unlock-starvation-handoff]]

Unlock() в Normal mode передаёт handoff=false. Что это значит для разбуженной горутины?
?
![[sync.Mutex/Unlock()#^unlock-slow-normal-wake]]
