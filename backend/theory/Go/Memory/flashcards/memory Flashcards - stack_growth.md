#flashcards/memory/stack_growth

С какого размера начинается стек горутины?
?
![[Stack growth#^sg-initial-size]]

Как компилятор Go обеспечивает проверку переполнения стека? Что вставляется в каждую функцию?
?
![[Stack growth#^09767d]]
![[Stack growth#^ea3b77]]
![[Stack growth#^sg-prologue-check]]

Что делает runtime.morestack()? Опиши шаги по порядку.
?
![[Stack growth#^sg-morestack-steps]]

Как Go управлял стеком до версии 1.4 и какая была проблема?
?
![[Stack growth#^sg-segmented-old]] + ![[Stack growth#^sg-hot-split]]

Что такое «hot split» в segmented stacks и почему это проблема?
?
![[Stack growth#^sg-hot-split]]

Что такое contiguous stacks (Go 1.4+) и какой у них трейдофф по сравнению с segmented?
?
![[Stack growth#^sg-contiguous]]

Каковы лимиты стека горутины? Начальный размер, максимум на 64-bit и 32-bit, шаг роста.
?
![[Stack growth#^sg-limits]]

Как Go корректирует указатели на стек при его росте? Что использует runtime?
?
![[Stack growth#^28a75e]]
![[Stack growth#^sg-stack-maps]]

Что произойдёт с uintptr при росте стека? Почему он не обновляется?
?
![[Stack growth#^sg-uintptr]]

При каком условии стек горутины сужается? В сколько раз?
?
![[Stack growth#^sg-shrink-threshold]]

Горутина имеет стек 8KB и использует 1.5KB. Что произойдёт со стеком?
?
![[Stack growth#^sg-shrink-threshold]] + ![[Stack growth#^sg-shrink-example]]
