#flashcards/channels-extra/select_priority

Данные одновременно в `highPriority` и `lowPriority`. Какой кейс выберет Go в `select`?
?
![[Приоритизация select#^prio-problem]]

Опиши три способа реализовать приоритет в select. Какой единственный даёт 100% гарантию?
?
![[Приоритизация select#^prio-nested-100]] + ![[Приоритизация select#^prio-top-select-prob]] + ![[Приоритизация select#^prio-duplicate-cases]]

Способ 1 — вложенный select. Опиши структуру кода и назови два его недостатка.
?
![[Приоритизация select#^prio-nested-100]]

Способ 2 — дополнительный select сверху. Почему он не даёт 100% гарантии?
?
![[Приоритизация select#^prio-top-select-prob]]

Способ 3 — дублирование кейсов. Как это меняет вероятности? Какова вероятность при 2 кейсах high и 1 кейсе low?
?
![[Приоритизация select#^prio-duplicate-cases]]
