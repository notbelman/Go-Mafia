#flashcards/mutex/example

G2, G3, G4 ждут лок в Normal mode. G1 делает Unlock, будит G2. В этот момент прилетает G5. Кто получит лок и почему?
?
![[пример#^example-normal-contention]]

G2 проиграл CAS горутине G5. Куда встаёт G2 в очереди?
?
![[пример#^example-normal-contention]]

G2 ждёт > 1ms. Что происходит с режимом мьютекса?
?
![[пример#^example-starvation-trigger]]

G5 делает Unlock в Starvation mode. Как лок передаётся G2? Что происходит с G6 которая прилетает в этот момент?
?
![[пример#^example-handoff]]

G2 получила лок через handoff. G3, G4, G6 в очереди. Выйдет ли G2 из Starvation mode?
?
![[пример#^example-starvation-stay]]

G6 последняя получает лок в Starvation mode. Что происходит дальше?
?
![[пример#^example-starvation-exit]]
