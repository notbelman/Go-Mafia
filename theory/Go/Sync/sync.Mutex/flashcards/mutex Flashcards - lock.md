#flashcards/mutex/lock

Что делает fast path в Lock()? Какая операция используется?
?
![[WORK-BASE/interviews/theory/Go/sync/sync.Mutex/Lock()#^lock-fast-path]]

Какие три условия должны выполняться чтобы горутина начала спиннить в Lock()?
?
![[WORK-BASE/interviews/theory/Go/sync/sync.Mutex/Lock()#^lock-spin-conditions]]

Сколько итераций спиннинга максимально в Lock()? Сколько циклов суммарно?
?
![[WORK-BASE/interviews/theory/Go/sync/sync.Mutex/Lock()#^lock-spin-conditions]]

Что происходит если все итерации спиннинга исчерпаны и CAS не удался?
?
![[WORK-BASE/interviews/theory/Go/sync/sync.Mutex/Lock()#^lock-sleep]]

Как ведёт себя Lock() в Starvation mode? Почему нет спиннинга?
?
![[WORK-BASE/interviews/theory/Go/sync/sync.Mutex/Lock()#^lock-starvation-path]]

Горутина проснулась в Normal mode после ожидания. Когда она устанавливает флаг starving?
?
![[WORK-BASE/interviews/theory/Go/sync/sync.Mutex/Lock()#^lock-wakeup-normal]]

Горутина в Normal mode проснулась, проиграла CAS новой горутине. Куда она встаёт в очереди?
?
![[WORK-BASE/interviews/theory/Go/sync/sync.Mutex/Lock()#^lock-wakeup-normal]]

Горутина получила лок через handoff (Starvation mode). При каких условиях она сбрасывает флаг starving?
?
![[WORK-BASE/interviews/theory/Go/sync/sync.Mutex/Lock()#^lock-starvation-exit]]
