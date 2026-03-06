#flashcards/gc/overview

Перечисли 4 фазы GC в Go по порядку. Какие из них STW, какие concurrent?
?
![[Обзор GC#Фаза 1: Sweep Termination (STW, ~10-30µs)]]
+
![[Обзор GC#Фаза 2 Mark (concurrent, ~25% CPU)]]
+
![[Обзор GC#Фаза 3 Mark Termination (STW, ~60-90µs)]]
+
![[Обзор GC#Фаза 4 Sweep (concurrent + lazy)]]

Какие конкретные длительности STW пауз в Go GC? Какая фаза сколько занимает?
?
![[Обзор GC#Фаза 1: Sweep Termination (STW, ~10-30µs)]] + ![[Обзор GC#Фаза 3: Mark Termination (STW, ~60-90µs)]]

Что происходит в фазе Sweep Termination по шагам?
?
![[Обзор GC#Фаза 1 Sweep Termination (STW, ~10-30µs)]]

Какой процент CPU отдаётся GC во время фазы Mark?
?
![[Обзор GC#Фаза 2 Mark (concurrent, ~25% CPU)]]

Какова формула heap goal — точка при которой триггерится GC?
?
![[Обзор GC#^gc-trigger-formula]]

Какие ещё два способа запустить GC помимо достижения heap goal?
?
![[Обзор GC#^gc-trigger-other]]

Что является root objects при обходе графа в фазе Mark?
?
![[Обзор GC#^phase-mark-roots]]

Зачем нужен Write barrier во время фазы Mark?
?
![[Обзор GC#^phase-mark-wb]]

Что произойдёт если горутина аллоцирует быстрее, чем GC успевает маркировать?
?
![[Обзор GC#^phase-mark-assist]]

Что случается с объектами у которых есть Finalizer — удаляются ли они в текущем цикле GC?
?
![[Обзор GC#^phase-mark-finalizers]]

Что происходит в фазе Mark Termination по шагам?
?
![[Обзор GC#Фаза 3 Mark Termination (STW, ~60-90µs)]]

Почему фаза Mark Termination дольше Sweep Termination (~60-90µs vs ~10-30µs)?
?
![[Обзор GC#Фаза 3 Mark Termination (STW, ~60-90µs)]]

Что значит "lazy sweep" в фазе 4? Как это работает?
?
![[Обзор GC#^phase-sweep-lazy]]

Что делает Scavenger в фазе Sweep?
?
![[Обзор GC#^phase-sweep-lazy]]
