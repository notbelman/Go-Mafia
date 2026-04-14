#flashcards/sync-map/promotion

Что такое promotion в sync.Map?
?
![[Promotion#^promotion-def]]

Когда срабатывает promotion? Точное условие.
?
![[Promotion#^promotion-trigger]]

Опиши состояние read, dirty, misses до и после promotion.
?
![[Promotion#^promotion-before-after]]

Что происходит со старым read после promotion?
?
![[Promotion#^promotion-gc]]

misses = 5, len(dirty) = 3. Сработал ли promotion раньше? Почему да/нет?
?
Да. Promotion срабатывает когда misses >= len(dirty). При 3 промахах на dirty из 3 элементов → misses(3) >= len(3) → promotion.
![[Promotion#^promotion-trigger]]

Что технически происходит при promotion?
?
![[Promotion#^promotion-mechanics]]
