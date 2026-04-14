#flashcards/gc/generational

В чём базовая идея generational GC? Какое наблюдение о поведении объектов лежит в основе?
?
![[Поколения (generational GC)#Идея]]

Что происходит с объектом, пережившим сборку мусора в generational GC?
?
![[Поколения (generational GC)#^gen-idea]]

Назови три поколения в типичном generational GC и частоту сборки каждого
?
![[Поколения (generational GC)#Идея]]

Опиши пошагово как объекты перемещаются между поколениями в generational GC
?
![[Поколения (generational GC)#Как работает]]

При каком условии GC заглядывает в старшее поколение?
?
![[Поколения (generational GC)#^gen-how-works]]

Почему в Go нет generational GC? Что заменяет «молодое поколение»?
?
![[Поколения (generational GC)#^go-no-gen]] + ![[Поколения (generational GC)#^stack-as-young-gen]]

Объясни аналогию: стек в Go ≈ молодое поколение. В чём она точна и в чём ограничена?
?
![[Поколения (generational GC)#^stack-as-young-gen]]

Почему Java нуждается в generational GC, а Go — нет? В чём архитектурная разница?
?
![[Поколения (generational GC)#^java-vs-go]]
