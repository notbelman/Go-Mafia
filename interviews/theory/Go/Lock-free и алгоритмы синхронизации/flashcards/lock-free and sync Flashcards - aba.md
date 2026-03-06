#flashcards/lock-free-and-sync/aba

Что такое ABA-проблема и почему она опасна для CAS?
?
![[ABA-проблема#^aba-def]]

CAS сравнивает адреса или содержимое памяти?
?
![[ABA-проблема#^aba-cas-compares-addr]]

Почему ABA-проблема не возникает в Go со стандартным GC?
?
![[ABA-проблема#^aba-why-go-safe]]

При каких условиях ABA-проблема может возникнуть даже в Go?
?
![[ABA-проблема#^aba-go-custom-alloc]]

Какие три класса решений существуют для ABA-проблемы?
?
![[ABA-проблема#^aba-solutions-list]]

Как работают hazard pointers? Опиши механизм.
?
![[ABA-проблема#^aba-hazard-pointers]]

Как tagged pointers решают ABA-проблему? Где хранится тег?
?
![[ABA-проблема#^aba-tagged-pointers]]

Почему tagged pointers зависят от архитектуры?
?
![[ABA-проблема#^aba-tagged-arch]]

Что такое "локальный GC для структуры данных" как решение ABA?
?
![[ABA-проблема#^aba-gc-solution]]
