#flashcards/channels-patterns/barrier

Что такое Barrier? В чём его отличие от WaitGroup?
?
![[Barrier#^barrier-idea]]

Из каких компонентов состоит Barrier? Почему два канала?
?
![[Barrier#^barrier-struct]]
![[Barrier#^barrier-before]]
![[Barrier#^barrier-after]]

Как работает Before() в Barrier? Кто разблокирует остальных?
?
![[Barrier#^barrier-before]]
![[Barrier#^barrier-key]]

Как работает After() в Barrier? Кто разблокирует остальных?
?
![[Barrier#^barrier-after]]
![[Barrier#^barrier-key]]

Почему каналы before и after создаются с буфером n, а не небуферизированными?
?
![[Barrier#^barrier-buf]]

Как используется Barrier? Покажи шаблон вызовов Before/After.
?
![[Barrier#^barrier-usage]]

Когда нужен Barrier? Приведи сценарии.
?
![[Barrier#^barrier-when]]
