#flashcards/channels-patterns/dynamic_select

Что такое динамический select? Чем ограничен обычный select?
?
![[Динамический select#^dynsel-problem]]

Как работает рекурсивная or-функция? Опиши механизм по шагам.
?
![[Динамический select#^dynsel-or-how]]

Зачем `orDone` передаётся в рекурсивный вызов `or(append(channels[2:], orDone)...)`?
?
![[Динамический select#^dynsel-or-done-purpose]]

Что вернёт or() если передать 0 каналов? 1 канал?
?
![[Динамический select#^dynsel-base-cases]]

Как работает reflect.Select? Что возвращает?
?
![[Динамический select#^dynsel-reflect]]

Сравни рекурсивную or-функцию и reflect.Select: скорость, сложность, возможности.
?
![[Динамический select#^dynsel-compare]]

Когда нужен динамический select? Приведи примеры.
?
![[Динамический select#^dynsel-when]]
