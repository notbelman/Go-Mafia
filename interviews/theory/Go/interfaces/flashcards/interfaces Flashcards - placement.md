#flashcards/interfaces/placement

Что такое producer-side и consumer-side расположение интерфейсов? Какой подход предпочитает Go коммьюнити?
?
![[Расположение интерфейсов#^producer-problem]]
![[Расположение интерфейсов#^consumer-advantages]]

Какая главная проблема producer-side интерфейса на примере Storage?
?
![[Расположение интерфейсов#^producer-problem]]

Каковы преимущества consumer-side интерфейса? Перечисли все.
?
![[Расположение интерфейсов#^consumer-advantages]]

Каковы трейдоффы producer-side: плюс и минусы?
?
![[Расположение интерфейсов#^producer-tradeoffs]]

Как инжектируется зависимость при consumer-side подходе? Что обеспечивает совместимость?
?
![[Расположение интерфейсов#^consumer-injection]]

Сравни producer vs consumer по четырём критериям: связанность, лишние методы, подмена реализации, изменение сигнатуры.
?
![[Расположение интерфейсов#^placement-comparison]]

Когда producer-side оправдан в stdlib? Приведи пример.
?
![[Расположение интерфейсов#^producer-exception]]
