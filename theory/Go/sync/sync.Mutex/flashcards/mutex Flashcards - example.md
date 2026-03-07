#flashcards/mutex/example

G2, G3, G4 ждут лок в Normal mode. G1 делает Unlock, будит G2. В этот момент прилетает G5. Кто получит лок и почему?
?
![[WORK-BASE/interviews/theory/Go/sync/sync.Mutex/пример#^example-normal-contention]]

G2 проиграл CAS горутине G5. Куда встаёт G2 в очереди?
?
![[WORK-BASE/interviews/theory/Go/sync/sync.Mutex/пример#^example-normal-contention]]

G2 ждёт > 1ms. Что происходит с режимом мьютекса?
?
![[WORK-BASE/interviews/theory/Go/sync/sync.Mutex/пример#^example-starvation-trigger]]

G5 делает Unlock в Starvation mode. Как лок передаётся G2? Что происходит с G6 которая прилетает в этот момент?
?
![[WORK-BASE/interviews/theory/Go/sync/sync.Mutex/пример#^example-handoff]]

G2 получила лок через handoff. G3, G4, G6 в очереди. Выйдет ли G2 из Starvation mode?
?
![[WORK-BASE/interviews/theory/Go/sync/sync.Mutex/пример#^example-starvation-stay]]

G6 последняя получает лок в Starvation mode. Что происходит дальше?
?
![[WORK-BASE/interviews/theory/Go/sync/sync.Mutex/пример#^example-starvation-exit]]
