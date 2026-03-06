#flashcards/channels-patterns/single_flight

Что такое Single Flight? Какую проблему решает?
?
![[Single Flight#^sf-problem]]

Опиши Thunder Herd problem. Почему это проблема для БД?
?
![[Single Flight#^sf-problem]]

Опиши пошагово как работает Do() в Single Flight для 1000 горутин с одним ключом.
?
![[Single Flight#^sf-flow]]

Из чего состоит структура call внутри Single Flight? Зачем там done-канал?
?
![[Single Flight#^sf-impl]]

Почему первая горутина тоже блокируется на `<-c.done`, если она сама создала call?
?
![[Single Flight#^sf-why-wait]]

Что происходит с записью в map после завершения fn()?
?
![[Single Flight#^sf-impl]]

Какие методы есть в `golang.org/x/sync/singleflight` помимо Do?
?
![[Single Flight#^sf-lib]]
