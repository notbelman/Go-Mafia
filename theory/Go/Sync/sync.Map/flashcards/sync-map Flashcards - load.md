#flashcards/sync-map/load

Опиши алгоритм Load() по шагам.
?
![[Load#^load-algorithm]]

Когда Load() работает полностью lock-free? Что именно происходит в этом случае?
?
![[Load#^load-lockfree-path]]

Как Load() использует misses? Когда инкрементирует?
?
![[Load#^load-misses]]

Ключ есть в read. Сколько атомарных операций записи делает Load()?
?
![[Load#^load-lockfree-path]]

Что произойдёт если вызвать Load() на несуществующий ключ при amended=true?
?
![[Load#^load-algorithm]]
