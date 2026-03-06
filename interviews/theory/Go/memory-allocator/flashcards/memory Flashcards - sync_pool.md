#flashcards/memory/sync_pool

Какова основная идея sync.Pool и как она снижает нагрузку на GC?
?
![[interviews/theory/Go/memory-allocator/sync.Pool#Идея]]

Какой API предоставляет sync.Pool? Что делает поле New?
?
![[sync.Pool#^pool-api]]

Почему после Get() из sync.Pool объект ОБЯЗАТЕЛЬНО нужно реинициализировать?
?
![[interviews/theory/Go/memory-allocator/sync.Pool#^pool-usage-reinit]]

Опиши жизненный цикл объекта в sync.Pool: два пути через Get().
?
![[interviews/theory/Go/memory-allocator/sync.Pool#^pool-lifecycle]]

Когда и почему GC забирает объекты из sync.Pool?
?
![[interviews/theory/Go/memory-allocator/sync.Pool#^pool-gc-eviction]]

Гарантирует ли sync.Pool, что Put-объект переживёт следующий цикл GC?
?
![[interviews/theory/Go/memory-allocator/sync.Pool#^pool-no-guarantee]]

Зачем делать типизированную обёртку над sync.Pool? Покажи паттерн.
?
![[interviews/theory/Go/memory-allocator/sync.Pool#^pool-typed-wrapper]]

В каких сценариях sync.Pool даёт существенный выигрыш? Приведи примеры.
?
![[interviews/theory/Go/memory-allocator/sync.Pool#^pool-use-cases]]

Является ли sync.Pool потокобезопасным? Можно ли вызывать Get/Put из разных горутин?
?
![[interviews/theory/Go/memory-allocator/sync.Pool#^5c8d35]]
