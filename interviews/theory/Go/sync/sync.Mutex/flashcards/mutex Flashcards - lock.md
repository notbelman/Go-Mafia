#flashcards/mutex/lock

Что делает fast path в Lock()? Какая операция используется?
?
![[Lock()#^lock-fast-path]]

Какие три условия должны выполняться чтобы горутина начала спиннить в Lock()?
?
![[Lock()#^lock-spin-conditions]]

Сколько итераций спиннинга максимально в Lock()? Сколько циклов суммарно?
?
![[Lock()#^lock-spin-conditions]]

Что происходит если все итерации спиннинга исчерпаны и CAS не удался?
?
![[Lock()#^lock-sleep]]

Как ведёт себя Lock() в Starvation mode? Почему нет спиннинга?
?
![[Lock()#^lock-starvation-path]]

Горутина проснулась в Normal mode после ожидания. Когда она устанавливает флаг starving?
?
![[Lock()#^lock-wakeup-normal]]

Горутина в Normal mode проснулась, проиграла CAS новой горутине. Куда она встаёт в очереди?
?
![[Lock()#^lock-wakeup-normal]]

Горутина получила лок через handoff (Starvation mode). При каких условиях она сбрасывает флаг starving?
?
![[Lock()#^lock-starvation-exit]]
