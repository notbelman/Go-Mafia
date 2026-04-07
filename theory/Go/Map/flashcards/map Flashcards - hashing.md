#flashcards/map/hashing

Опиши как 64-битный хеш разбивается на tophash и bucket index
?
![[хеширование и поиск bucket#^hash-bits-split]]

Что такое tophash? Откуда берётся?
?
![[хеширование и поиск bucket#^hash-tophash-def]]

Как из хеша вычисляется номер bucket? Почему именно так, а не деление?
?
![[хеширование и поиск bucket#^hash-bucket-index-def]]

B=2, hash("foo") = 0xB4...73. Какой bucket? Покажи вычисление.
?
![[хеширование и поиск bucket#^hash-example-b2]]

Опиши алгоритм поиска значения по ключу на уровне битовых операций
?
![[хеширование и поиск bucket#^hash-lookup-code]]

Какова эффективность tophash как фильтра?
?
![[хеширование и поиск bucket#^hash-tophash-efficiency]]

Почему tophash ускоряет поиск внутри bucket? Что он заменяет?
?
![[хеширование и поиск bucket#^hash-tophash-efficiency]]

Как вычисляется tophash из хеша? Конкретная операция.
?
![[хеширование и поиск bucket#^hash-lookup-code]]
