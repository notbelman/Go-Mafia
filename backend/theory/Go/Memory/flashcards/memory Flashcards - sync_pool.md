#flashcards/memory/sync_pool

Какова основная идея sync.Pool и как она снижает нагрузку на GC?
?
![[Memory/sync.Pool#Идея]]

Какой API предоставляет sync.Pool? Что делает поле New?
?
![[Sync/sync.Pool#^pool-api]]

Почему после Get() из sync.Pool объект ОБЯЗАТЕЛЬНО нужно реинициализировать?
?
![[Memory/sync.Pool#^pool-usage-reinit]]

Опиши жизненный цикл объекта в sync.Pool: два пути через Get().
?
![[Memory/sync.Pool#^pool-lifecycle]]

Когда и почему GC забирает объекты из sync.Pool?
?
![[Memory/sync.Pool#^pool-gc-eviction]]

Гарантирует ли sync.Pool, что Put-объект переживёт следующий цикл GC?
?
![[Memory/sync.Pool#^pool-no-guarantee]]

Зачем делать типизированную обёртку над sync.Pool? Покажи паттерн.
?
![[Memory/sync.Pool#^pool-typed-wrapper]]

В каких сценариях sync.Pool даёт существенный выигрыш? Приведи примеры.
?
![[Memory/sync.Pool#^pool-use-cases]]

Является ли sync.Pool потокобезопасным? Можно ли вызывать Get/Put из разных горутин?
?
![[Memory/sync.Pool#^5c8d35]]
