#flashcards/gc/reference_counting

Опиши механизм reference counting: что хранит объект, что происходит при копировании/удалении ссылки, когда объект освобождается?
?
![[Reference counting#^rc-idea]]

В чём главный плюс reference counting по сравнению с tracing GC?
?
![[Reference counting#^rc-pros]]

Какой overhead несёт reference counting на каждую операцию с указателем?
?
![[Reference counting#^rc-overhead]]

Почему reference counting требует синхронизации в многопоточной среде? Как это решается?
?
![[Reference counting#^rc-sync]]

Что такое каскадное синхронное освобождение? Приведи конкретный пример с числами.
?
![[Reference counting#^rc-cascade]]

Почему каскадное освобождение — проблема для latency?
?
![[Reference counting#^rc-cascade]]

Почему reference counting не может освободить циклические ссылки? Покажи на примере двусвязного списка.
?
![[Reference counting#^rc-cycles-problem]]

Как C++ решает проблему циклических ссылок? Какой ценой?
?
![[Reference counting#^rc-cycles-solution]]

Почему Go не выбрал reference counting как основной механизм GC?
?
![[Reference counting#^rc-why-not-go]]

Чем семантически отличается reference counting от tracing GC — что каждый из них ищет?
?
![[Reference counting#^rc-idea]] + ![[Tracing (базовый STW)#^tracing-idea]]
