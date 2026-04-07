#flashcards/sync-primitive-supple/false_sharing

Что такое кэш-линия и какой у неё размер?
?
![[False_Sharing#^fs-cache-line-size]]

Что такое false sharing? Почему шардированный счётчик без padding почти не быстрее одного атомика?
?
![[False_Sharing#^fs-definition]]

Сколько atomic.Int64 шардов помещается в одну кэш-линию 64 байта? Почему это проблема?
?
![[False_Sharing#^fs-problem-math]]

Опиши механизм инвалидации при false sharing: что происходит когда ядро 0 инкрементирует shard[0]?
?
![[False_Sharing#^fs-invalidation-mechanism]]

Как решить false sharing через padding? Напиши структуру PaddedCounter с правильным размером padding.
?
![[False_Sharing#^fs-padding-solution]]

Почему padding 128 байт может быть лучше чем 64 байта?
?
![[False_Sharing#^fs-padding-128]]

Какой результат бенчмарка для четырёх подходов: Mutex / Atomic / Шарды без padding / Шарды + padding 64?
?
![[False_Sharing#^fs-benchmark]]

В чём разница между true sharing и false sharing? Можно ли устранить true sharing padding'ом?
?
![[False_Sharing#^fs-true-sharing]] + ![[False_Sharing#^fs-false-sharing-def]]

Где ещё в стандартной библиотеке Go используется паттерн per-P + padding?
?
![[False_Sharing#^fs-solution-summary]]

В каком реальном контексте (приложение / библиотека) актуален шардированный счётчик с padding?
?
![[False_Sharing#^fs-sharded-context]]
