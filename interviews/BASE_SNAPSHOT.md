# BASE SNAPSHOT

## Дата и методика
- Дата генерации: `2026-02-26 13:12:10`
- Источники:
  - `/Users/caoguojun0/Documents/Obsidian/main/interviews/theory`
  - `/Users/caoguojun0/Documents/Obsidian/main/interviews/questions/theory`
  - `/Users/caoguojun0/Documents/Obsidian/main/interviews/questions/tasks`
  - `/Users/caoguojun0/Documents/Obsidian/main/interviews/questions/hr`
  - `/Users/caoguojun0/Documents/Obsidian/main/interviews/questions/by-companies`
- Методика:
  - `что разобрано` определяется по именам `.md` внутри подпапок.
  - `flashcards_count` считается по подпапкам с именем `flashcards`.
  - `flashcards_coverage = yes`, если в подтеме есть хотя бы 1 `.md` в `flashcards`; иначе `no`.
  - В счётчики и покрытие не входят `.png`, `.canvas`, `.DS_Store`.

## Общая карта базы

| Раздел | md_count |
|---|---:|
| prompts | 9 |
| questions | 290 |
| theory | 1231 |

| Тип файлов | count |
|---|---:|
| `.md` | 1532 |
| `.png` | 85 |
| `.canvas` | 1 |

Дерево (2 уровня):
```text
main/interviews
├── prompts
├── questions
│   ├── by-companies
│   ├── hr
│   ├── tasks
│   └── theory
└── theory
    ├── Docker
    ├── Go
    ├── PostgreSQL
    ├── Redis
    ├── Rust
    ├── grpc
    ├── k8s
    ├── sd
    ├── Брокеры сообщений
    ├── Микросервисы и монолиты
    ├── Синхронные интеграции
    ├── архитектура кода
    └── курсы и доп материалы
```

## Theory: домены и объём

| # | Домен | md_count |
|---:|---|---:|
| 1 | Go | 700 |
| 2 | sd | 182 |
| 3 | PostgreSQL | 116 |
| 4 | Rust | 62 |
| 5 | Брокеры сообщений | 39 |
| 6 | Redis | 35 |
| 7 | Синхронные интеграции | 30 |
| 8 | Микросервисы и монолиты | 26 |
| 9 | grpc | 17 |
| 10 | Docker | 9 |
| 11 | архитектура кода | 9 |
| 12 | k8s | 3 |
| 13 | курсы и доп материалы | 3 |

## Deep Dive: Go

| Подтема                             | md_count | flashcards_count | flashcards_coverage | что разобрано                                                                                                                                                                                                                                                                          |
| ----------------------------------- | -------: | ---------------: | ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| context-iterators                   |       15 |               14 | yes                 | Context ошибки и правила, Context, Graceful Shutdown, context AfterFunc, context Background и TODO, context WithCancel, context WithDeadline, context WithTimeout, context WithValue, context WithoutCancel ... (+5)                                                                   |
| Data Race vs Race Condition         |        5 |                5 | yes                 | Data Race vs Race Condition, Data Race, Happens-Before в Go, Race Condition, Race Detector (-race)                                                                                                                                                                                     |
| defer                               |        4 |                0 | no                  | defer под капотом, Почему Mutex.Unlock() лучше не в defer, основы, продвинутое                                                                                                                                                                                                         |
| errors                              |       14 |               14 | yes                 | Defer и паники, Multierror, Sentinel vs кастомный тип vs поведение, Sentinel ошибки, errors.Is и errors.As, Восстановимые и невосстановимые ошибки, Игнорирование ошибок и ошибки из defer, Интерфейс error, Оборачивание ошибок, Особенности паник ... (+4)                         |
| functions                           |       17 |               17 | yes                 | Calling conventions и Go ABI, Defer аргументы и ловушки, Defer и именованные возвращаемые, Defer механика и порядок, Inlining функций, Анонимные функции и variadic, Аппаратный стек и вызов функций, Декоратор и композиция, Замыкание, Именованные возвращаемые значения ... (+7) |
| generics-reflect                    |       16 |               15 | yes                 | Constraints на методы и поля, Constraints, Mixin и CRTP, Type assertion в дженериках, Type inference и параметры типов, Возможности рефлексии, Зачем дженерики, Когда использовать и цена дженериков, Недостатки рефлексии, Обобщённая фабрика и декоратор ... (+6)                   |
| interfaces                          |       19 |               19 | yes                 | Best practices, Embedding и реализация интерфейса, Type assertion и type switch, any vs interface{}, eface, iface структура, nil интерфейса, Вызов метода на nil, Дженерики vs Интерфейсы, Диспетчеризация и девиртуализация ... (+9)                                               |
| Lock-free и алгоритмы синхронизации |        9 |                9 | yes                 | ABA-проблема, RCU, Акторная модель, Алгоритмы синхронизации списков, Аномалии конкурентных списков, Закон Амдала, Очередь Майкла-Скотта, Стек Трайбера, Шардированная мапа                                                                                                           |
| map                                 |       22 |               22 | yes                 | Evacuation, HashDoS — атака на мапу, Map = указатель, Overflow buckets, Resize x2, Same size rehash, Set в Go, Swiss Tables, bucket (bmap), iteration order ... (+12)                                                                                                                  |
| memory-allocator                    |       25 |               26 | yes                 | Atomic и выравнивание, Escape analysis - что это и зачем, Inlining и escape, Stack growth, TCMalloc - почему Go так делает, Happens-before через atomic, MESI vs барьеры, Reordering инструкций, Барьеры памяти, sync.Pool ... (+15)                                                  |
| runtime                             |       29 |               28 | yes                 | Concurrency vs Parallelism, Finalizers, GC Pacer, GOGC и GOMEMLIMIT, Lazy allocation и RSS vs VSS, Mark Assist, Reference counting, Tracing (базовый STW), Tri-color marking, Write barrier ... (+19)                                                                                 |
| slice-array-cards-v2                |       23 |               23 | yes                 | Array аллокация и копирование, Array основы, Bound check elimination, Full slice expression, GC и slice, Range подводные камни, Slice copy, Slice vs Array, Slice аллокация стек и хип, Slice от slice баги ... (+13)                                                                  |
| str                                 |       14 |               14 | yes                 | Range по строке, Rune и UTF-8, Unsafe string to byte, byte конверсия, interning, len и подстроки, strings Builder, Виды строк, Как Go понимает сколько байт в символе, Кодировки ASCII UTF-32 ... (+4)                                                                                |
| structs                             |       15 |               15 | yes                 | Closer паттерн, DOD Data Oriented Design, Defer и структуры, Functional Options, Type alias vs Type definition, Value vs Pointer receiver, Встраивание когда и когда нет, Встраивание типов, Выравнивание инструменты и практика, Выравнивание структур ... (+5)                       |
| sync                                |       57 |               56 | yes                 | Go Memory Model и Memory Barriers, Atomic vs Mutex, Memory Ordering, atomic.Value, sync - atomic, Почему НЕ atomic везде, CAS и CAS loop, Deadlock, False Sharing, Livelock и Starvation ... (+47)                                                                                     |
| unsafe                              |        4 |                4 | yes                 | Sizeof,Alignof,Offsetof, unsafe — практические use cases, unsafe.Pointer и uintptr, Правила unsafe.Pointer                                                                                                                                                                             |
| Каналы                              |       69 |               54 | yes                 | Deadlock в каналах, Race condition в каналах, Базовый API и range, ДОПОЛНЕНИЯ к существующим, Неблокирующие операции, Нюансы каналов, Приоритизация select, Утечки горутин и правила закрытия, gopark() и goready(), Disable case ... (+59)                                           |
| ооп                                 |        7 |                0 | no                  | D — Dependency Inversion, I — Interface Segregation, L — Liskov Substitution, O — Open Closed, S — Single Responsibility, Композиция vs Наследование, ООП в Go — общий ответ                                                                                                           |

Полный список `что разобрано` по Go-подтемам (basename):
- `Data Race vs Race Condition` (5): Data Race vs Race Condition, Data Race, Happens-Before в Go, Race Condition, Race Detector (-race)
- `Lock-free и алгоритмы синхронизации` (9): ABA-проблема, RCU, Акторная модель, Алгоритмы синхронизации списков, Аномалии конкурентных списков, Закон Амдала, Очередь Майкла-Скотта, Стек Трайбера, Шардированная мапа
- `context-iterators` (15): Context ошибки и правила, Context, Graceful Shutdown, context AfterFunc, context Background и TODO, context WithCancel, context WithDeadline, context WithTimeout, context WithValue, context WithoutCancel, errgroup с контекстами, Итераторы горутины и ошибки, Итераторы применение, Итераторы, Оборачивание функций без контекста
- `defer` (4): defer под капотом, Почему Mutex.Unlock() лучше не в defer, основы, продвинутое
- `errors` (14): Defer и паники, Multierror, Sentinel vs кастомный тип vs поведение, Sentinel ошибки, errors.Is и errors.As, Восстановимые и невосстановимые ошибки, Игнорирование ошибок и ошибки из defer, Интерфейс error, Оборачивание ошибок, Особенности паник, Паника и recover, Правила обработки ошибок, Способы сигнализации об ошибках, Стектрейс ошибки
- `functions` (17): Calling conventions и Go ABI, Defer аргументы и ловушки, Defer и именованные возвращаемые, Defer механика и порядок, Inlining функций, Анонимные функции и variadic, Аппаратный стек и вызов функций, Декоратор и композиция, Замыкание, Именованные возвращаемые значения, Каррирование и ленивые вычисления, Ограничения функций в Go, Рекурсия и мемоизация, ФВП и предикаты, Функции основы, Функции первого класса, Чистые функции
- `generics-reflect` (16): Constraints на методы и поля, Constraints, Mixin и CRTP, Type assertion в дженериках, Type inference и параметры типов, Возможности рефлексии, Зачем дженерики, Когда использовать и цена дженериков, Недостатки рефлексии, Обобщённая фабрика и декоратор, Обобщённые структуры и type definitions, Ограничения дженериков, Практические кейсы рефлексии, Рефлексия vs интроспекция, Теги структур, Три свойства рефлексии
- `interfaces` (19): Best practices, Embedding и реализация интерфейса, Type assertion и type switch, any vs interface{}, eface, iface структура, nil интерфейса, Вызов метода на nil, Дженерики vs Интерфейсы, Диспетчеризация и девиртуализация, Иммутабельность интерфейсов, Копирование и ловушки с типами, Кэш itab, Мономорфизация, Полиморфизм и утиная типизация, Расположение интерфейсов, Статический и динамический тип, Стоимость type assertion, Что такое интерфейс
- `map` (22): Evacuation, HashDoS — атака на мапу, Map = указатель, Overflow buckets, Resize x2, Same size rehash, Set в Go, Swiss Tables, bucket (bmap), iteration order, nil vs empty map, Итерация и мутация, Ключи map, Метод открытой адресации, Метод цепочек, Память и утечки map, Хэш-таблицы теория, не потокобезопасна, нельзя взять адрес value, операции чтения и вставки, структура hmap, хеширование и поиск bucket
- `memory-allocator` (25): Atomic и выравнивание, Escape analysis - что это и зачем, Inlining и escape, Stack growth, TCMalloc - почему Go так делает, Happens-before через atomic, MESI vs барьеры, Reordering инструкций, Барьеры памяти, sync.Pool, Алгоритмы аллокации (база), Арены (experimental), Выравнивание (alignment), Классы размеров (size classes), Куча что это, Направление роста стека и кучи, Организация памяти кучи (арены - страницы - спаны), Почему стек ОС-треда 1-8 MB, Практические приёмы уменьшения аллокаций, Пример аллокации, Стек vs Куча, Стек пример, Стек, Три уровня аллокатора (mcache - mcentral - mheap), Что вызывает escape
- `runtime` (29): Concurrency vs Parallelism, Finalizers, GC Pacer, GOGC и GOMEMLIMIT, Lazy allocation и RSS vs VSS, Mark Assist, Reference counting, Tracing (базовый STW), Tri-color marking, Write barrier, Обзор GC, Поколения (generational GC), Что такое мусор и когда можно не собирать, G (Goroutine), GMP обзор, LRQ и GRQ, M (Machine), P (Processor), handoff, netpoller, preemption, sysmon, work sharing vs work stealing, work stealing, Внутреннее устройство очередей, Горутины — что это и зачем, Нюансы горутин, Паники и горутины, как работают вместе
- `slice-array-cards-v2` (23): Array аллокация и копирование, Array основы, Bound check elimination, Full slice expression, GC и slice, Range подводные камни, Slice copy, Slice vs Array, Slice аллокация стек и хип, Slice от slice баги, Slice передача в функцию, Slice структура, Slice утечки памяти, Slice — не comparable, Subslicing нарезка, append внутри функции — НЕ видно снаружи, append, len vs cap, nil slice vs empty slice, Как сделать append видимым, Массив vs Слайс vs Мапа, Создание slice, Удаление и очистка slice
- `str` (14): Range по строке, Rune и UTF-8, Unsafe string to byte, byte конверсия, interning, len и подстроки, strings Builder, Виды строк, Как Go понимает сколько байт в символе, Кодировки ASCII UTF-32, Сравнение строк, Строки утечки памяти, иммутабельность, структура
- `structs` (15): Closer паттерн, DOD Data Oriented Design, Defer и структуры, Functional Options, Type alias vs Type definition, Value vs Pointer receiver, Встраивание когда и когда нет, Встраивание типов, Выравнивание инструменты и практика, Выравнивание структур, Методы и ресиверы, Ограничения методов и unsafe cast, Порядок полей и сравнение, Пустые структуры, Структуры основы
- `sync` (57): Go Memory Model и Memory Barriers, Atomic vs Mutex, Memory Ordering, atomic.Value, sync - atomic, Почему НЕ atomic везде, CAS и CAS loop, Deadlock, False Sharing, Livelock и Starvation, WaitGroup копирование и нюансы, Практические ошибки с мьютексами, sync.Cond, Cтруктура, Delete, Len, Load(), Promotion, Range, Store(), amended, dirtyLocked, entry состояния, sync.Map vs map + RWMutex — Когда Что Использовать, sync.Map, Все методы, Пример ебнутый, Структура, Lock(), Unlock(), sema (семафор), state, sync.Mutex.TryLock (Go 1.18+), Два режима (с Go 1.9), Структура, пример, sync.Mutex vs sync.RWMutex, sync.Once, sync.Pool, Lock(), RLock(), RUnlock(), TryRLock TryLock (Go 1.18+), Unlock(), sync.RWMutex, пример, sync.WaitGroup, sync.Wg пример, CAS паттерны, Recursive и Timed mutex, Spin lock, Spurious wakeup, Ticket lock, True sharing и False sharing, Мьютекс Петерсона, Примитивный мьютекс, Семафор
- `unsafe` (4): Sizeof,Alignof,Offsetof, unsafe — практические use cases, unsafe.Pointer и uintptr, Правила unsafe.Pointer
- `Каналы` (69): Deadlock в каналах, Race condition в каналах, Базовый API и range, ДОПОЛНЕНИЯ к существующим, Неблокирующие операции, Нюансы каналов, Приоритизация select, Утечки горутин и правила закрытия, gopark() и goready(), Disable case, Nil-канал, Механика операций, Поведение в select, select - default, select - Специфические состояния и паттерны, select - Таблица поведения, select - алгоритм работы, select, Ticker и Timer утечки и GC, Ticker, Timer, time Tick и time After, waitq и sudog — пример (recvq), waitq и sudog — пример (sendq), waitq и sudog, Lock-free Fast Paths, 1 - буфер не пуст, никто не ждёт, 2 - буфер полон + sender ждёт (Handoff), 3 - буфер пуст + sender ждёт (Direct Copy,Bypass), 4 - буфер пуст, никто не ждёт, 1 - буфер не полон, никто не ждёт, 2 - буфер пуст + receiver ждёт (Direct Copy), 3 - буфер полон, никто не ждёт, Аллокация памяти, Буферизированные каналы, Внутреннее устройство (hchan), Кольцевой буфер, Закрытие канала, Модели конкурентности, Направленные каналы (Directional Channels), Направленные каналы (Directional Channels), Receive (-ch), Send (ch - v), Внутреннее устройство (hchan), Небуферизированные каналы, Паттерны, Поля структуры hchan, Внутреннее устройство (hchan), Когда использовать + ошибки, Поведение в select, Поведение операций, Семантика синхронизации, Что это, Barrier, Bridge, Done channel, Error Group, Fan-In, Fan-Out и Tee, Generator, Graceful shutdown, Moving Letter, Or-done channel, Pipeline, Promise и Future, Rate Limiter, Single Flight, Transform и Filter, Динамический select
- `ооп` (7): D — Dependency Inversion, I — Interface Segregation, L — Liskov Substitution, O — Open Closed, S — Single Responsibility, Композиция vs Наследование, ООП в Go — общий ответ

### GMP / Scheduler
- `md_count`: 16, `flashcards_count`: 16
- Что разобрано: G (Goroutine), GMP обзор, LRQ и GRQ, M (Machine), P (Processor), handoff, netpoller, preemption, sysmon, work sharing vs work stealing, work stealing, Внутреннее устройство очередей, Горутины — что это и зачем, Нюансы горутин, Паники и горутины, как работают вместе

| Элемент | Покрытие | Подтверждение |
|---|---|---|
| G/M/P | есть | G (Goroutine), GMP обзор, M (Machine), P (Processor) |
| LRQ/GRQ | есть | LRQ и GRQ, GMP Flashcards - lrq grq, Внутреннее устройство очередей |
| work stealing/sharing | есть | GMP Flashcards - work stealing, work sharing vs work stealing, work stealing |
| netpoller | есть | GMP Flashcards - netpoller, netpoller |
| sysmon | есть | GMP Flashcards - sysmon, sysmon |
| preemption | есть | GMP Flashcards - preemption, preemption |
| handoff | есть | GMP Flashcards - handoff, handoff |
| panics | есть | GMP Flashcards - panics goroutines, Паники и горутины |

### Каналы
- `md_count`: 69, `flashcards_count`: 54
- Что разобрано: Deadlock в каналах, Race condition в каналах, Базовый API и range, ДОПОЛНЕНИЯ к существующим, Неблокирующие операции, Нюансы каналов, Приоритизация select, Утечки горутин и правила закрытия, gopark() и goready(), Disable case, Nil-канал, Механика операций, Поведение в select, select - default, select - Специфические состояния и паттерны, select - Таблица поведения, select - алгоритм работы, select, Ticker и Timer утечки и GC, Ticker, Timer, time Tick и time After, waitq и sudog — пример (recvq), waitq и sudog — пример (sendq), waitq и sudog, Lock-free Fast Paths, 1 - буфер не пуст, никто не ждёт, 2 - буфер полон + sender ждёт (Handoff), 3 - буфер пуст + sender ждёт (Direct Copy,Bypass), 4 - буфер пуст, никто не ждёт, 1 - буфер не полон, никто не ждёт, 2 - буфер пуст + receiver ждёт (Direct Copy), 3 - буфер полон, никто не ждёт, Аллокация памяти, Буферизированные каналы, Внутреннее устройство (hchan), Кольцевой буфер, Закрытие канала, Модели конкурентности, Направленные каналы (Directional Channels), Направленные каналы (Directional Channels), Receive (-ch), Send (ch - v), Внутреннее устройство (hchan), Небуферизированные каналы, Паттерны, Поля структуры hchan, Внутреннее устройство (hchan), Когда использовать + ошибки, Поведение в select, Поведение операций, Семантика синхронизации, Что это, Barrier, Bridge, Done channel, Error Group, Fan-In, Fan-Out и Tee, Generator, Graceful shutdown, Moving Letter, Or-done channel, Pipeline, Promise и Future, Rate Limiter, Single Flight, Transform и Filter, Динамический select

### sync/atomic/mutex/RWMutex/sync.Map
- `md_count`: 57, `flashcards_count`: 56
- Что разобрано: Go Memory Model и Memory Barriers, Atomic vs Mutex, Memory Ordering, atomic.Value, sync - atomic, Почему НЕ atomic везде, CAS и CAS loop, Deadlock, False Sharing, Livelock и Starvation, WaitGroup копирование и нюансы, Практические ошибки с мьютексами, sync.Cond, Cтруктура, Delete, Len, Load(), Promotion, Range, Store(), amended, dirtyLocked, entry состояния, sync.Map vs map + RWMutex — Когда Что Использовать, sync.Map, Все методы, Пример ебнутый, Структура, Lock(), Unlock(), sema (семафор), state, sync.Mutex.TryLock (Go 1.18+), Два режима (с Go 1.9), Структура, пример, sync.Mutex vs sync.RWMutex, sync.Once, sync.Pool, Lock(), RLock(), RUnlock(), TryRLock TryLock (Go 1.18+), Unlock(), sync.RWMutex, пример, sync.WaitGroup, sync.Wg пример, CAS паттерны, Recursive и Timed mutex, Spin lock, Spurious wakeup, Ticket lock, True sharing и False sharing, Мьютекс Петерсона, Примитивный мьютекс, Семафор

### Memory allocator / Stack-Heap / Escape / GC
- `md_count`: 37, `flashcards_count`: 38
- Что разобрано: Atomic и выравнивание, Escape analysis - что это и зачем, Inlining и escape, Stack growth, TCMalloc - почему Go так делает, Happens-before через atomic, MESI vs барьеры, Reordering инструкций, Барьеры памяти, sync.Pool, Алгоритмы аллокации (база), Арены (experimental), Выравнивание (alignment), Классы размеров (size classes), Куча что это, Направление роста стека и кучи, Организация памяти кучи (арены - страницы - спаны), Почему стек ОС-треда 1-8 MB, Практические приёмы уменьшения аллокаций, Пример аллокации, Стек vs Куча, Стек пример, Стек, Три уровня аллокатора (mcache - mcentral - mheap), Что вызывает escape, Finalizers, GC Pacer, GOGC и GOMEMLIMIT, Lazy allocation и RSS vs VSS, Mark Assist, Reference counting, Tracing (базовый STW), Tri-color marking, Write barrier, Обзор GC, Поколения (generational GC), Что такое мусор и когда можно не собирать

### map
- `md_count`: 22, `flashcards_count`: 22
- Что разобрано: Evacuation, HashDoS — атака на мапу, Map = указатель, Overflow buckets, Resize x2, Same size rehash, Set в Go, Swiss Tables, bucket (bmap), iteration order, nil vs empty map, Итерация и мутация, Ключи map, Метод открытой адресации, Метод цепочек, Память и утечки map, Хэш-таблицы теория, не потокобезопасна, нельзя взять адрес value, операции чтения и вставки, структура hmap, хеширование и поиск bucket

### slice-array
- `md_count`: 23, `flashcards_count`: 23
- Что разобрано: Array аллокация и копирование, Array основы, Bound check elimination, Full slice expression, GC и slice, Range подводные камни, Slice copy, Slice vs Array, Slice аллокация стек и хип, Slice от slice баги, Slice передача в функцию, Slice структура, Slice утечки памяти, Slice — не comparable, Subslicing нарезка, append внутри функции — НЕ видно снаружи, append, len vs cap, nil slice vs empty slice, Как сделать append видимым, Массив vs Слайс vs Мапа, Создание slice, Удаление и очистка slice

### errors
- `md_count`: 14, `flashcards_count`: 14
- Что разобрано: Defer и паники, Multierror, Sentinel vs кастомный тип vs поведение, Sentinel ошибки, errors.Is и errors.As, Восстановимые и невосстановимые ошибки, Игнорирование ошибок и ошибки из defer, Интерфейс error, Оборачивание ошибок, Особенности паник, Паника и recover, Правила обработки ошибок, Способы сигнализации об ошибках, Стектрейс ошибки

### interfaces
- `md_count`: 19, `flashcards_count`: 19
- Что разобрано: Best practices, Embedding и реализация интерфейса, Type assertion и type switch, any vs interface{}, eface, iface структура, nil интерфейса, Вызов метода на nil, Дженерики vs Интерфейсы, Диспетчеризация и девиртуализация, Иммутабельность интерфейсов, Копирование и ловушки с типами, Кэш itab, Мономорфизация, Полиморфизм и утиная типизация, Расположение интерфейсов, Статический и динамический тип, Стоимость type assertion, Что такое интерфейс

### generics-reflect
- `md_count`: 16, `flashcards_count`: 15
- Что разобрано: Constraints на методы и поля, Constraints, Mixin и CRTP, Type assertion в дженериках, Type inference и параметры типов, Возможности рефлексии, Зачем дженерики, Когда использовать и цена дженериков, Недостатки рефлексии, Обобщённая фабрика и декоратор, Обобщённые структуры и type definitions, Ограничения дженериков, Практические кейсы рефлексии, Рефлексия vs интроспекция, Теги структур, Три свойства рефлексии

## Deep Dive: PostgreSQL

| Подтема | md_count | flashcards_count | что разобрано |
|---|---:|---:|---|
| JOIN | 13 | 0 | CROSS JOIN, FULL OUTER JOIN, INNER JOIN, LATERAL JOIN, LEFT JOIN, LEFT LATERAL vs CROSS LATERAL, ON vs WHERE в LEFT JOIN, RIGHT JOIN, NULL в условиях JOIN, USING и NATURAL ... (+3) |
| replication-cards | 14 | 0 | CDC и libslave, Eventual Consistency, Masterless (без ведущих узлов), Multi-Leader (Master-Master), Physical vs Logical репликация, Replication Lag и аномалии асинхронной репликации, SBR vs RBR (Statement vs Row Based Replication), Single-Leader (Master-Slave), Strong Consistency, Tunable Consistency (Cassandra) ... (+4) |
| Vacuum | 2 | 0 | VACUUM FULL, VACUUM |
| Индексы | 27 | 0 | CREATE INDEX CONCURRENTLY, UPDATE, DELETE (Мертвые строки), Адрес строки (TID CTID), Как индекс использует адрес, Что это - Структура страницы, REINDEX, Что это, Expression индексы (functional), Покрывающие индексы (covering index Index-Only Scan), Составные индексы (multi-column) ... (+17) |
| Константы | 4 | 0 | FOREIGN KEY действия, Primary Key в PostgreSQL, Модификаторы, Общая табличка |
| Оптимизация запросов | 17 | 0 | Cost, EXPLAIN и EXPLAIN ANALYZE, Как читать, ANTI JOIN — нет ни одного совпадения, Hash Join, Materialize, Merge Join, Nested Loop Join, SEMI JOIN — есть ли хоть одно совпадение, Bitmap Heap Scan ... (+7) |
| Партиционирование | 9 | 0 | pg partman, Вертикальное партиционирование, Партиционирование, Composite Partitioning, Hash Partitioning, List Partitioning, Range Partitioning, Термины партиционирования, Типы партиционирования |
| Транзакции | 18 | 0 | 2PL, MVCC, Dirty Read (грязное чтение), Lost Update (потерянное обновление), Non-Repeatable Read (неповторяющееся чтение), Phantom Read (фантомное чтение), Зачем, Табличка, MVCC (Multi-Version Concurrency Control), Зачем ... (+8) |
| Шардирование | 9 | 0 | Consistent Hashing, Virtual Buckets (виртуальное шардирование), Генерация ID в шардированных системах, Перебалансировка шардов (переналивка), Рандеву хеширование (Rendezvous, HRW), Роутинг запросов к шардам, Грабли шардирования, Стратегии роутинга шардов, Шардирование |

### JOIN
- `md_count`: 13, `flashcards_count`: 0
- Что разобрано: CROSS JOIN, FULL OUTER JOIN, INNER JOIN, LATERAL JOIN, LEFT JOIN, LEFT LATERAL vs CROSS LATERAL, ON vs WHERE в LEFT JOIN, RIGHT JOIN, NULL в условиях JOIN, USING и NATURAL, Почему диаграммы Венна врут, Производительность JOIN, Условие ON

### Индексы
- `md_count`: 27, `flashcards_count`: 0
- Что разобрано: CREATE INDEX CONCURRENTLY, UPDATE, DELETE (Мертвые строки), Адрес строки (TID CTID), Как индекс использует адрес, Что это - Структура страницы, REINDEX, Что это, Expression индексы (functional), Покрывающие индексы (covering index Index-Only Scan), Составные индексы (multi-column), Уникальные индексы и constraints, Частичные индексы (partial index), Издержки индексов, Индекс, Какие бывают индексы в PostgreSQL - сводка, Помогает ли индекс для GROUP BY, Селективность индекса, B-Tree vs LSM-Tree, B-tree индекс - внутреннее устройство, B-tree индекс - что это, Balanced tree vs Binary tree, Как работает поиск, Когда использовать и не использовать, Структура, GIN индекс (Generalized Inverted Index), GiST индекс (Generalized Search Tree), Hash индекс

### Транзакции
- `md_count`: 18, `flashcards_count`: 0
- Что разобрано: 2PL, MVCC, Dirty Read (грязное чтение), Lost Update (потерянное обновление), Non-Repeatable Read (неповторяющееся чтение), Phantom Read (фантомное чтение), Зачем, Табличка, MVCC (Multi-Version Concurrency Control), Зачем, Ключевое, Под капотом, Snapshot и Visibility, Примеры, XID (Transaction ID), Уровни изоляции PostgreSQL, Синтаксис, Транзакция и ACID

### Оптимизация запросов
- `md_count`: 17, `flashcards_count`: 0
- Что разобрано: Cost, EXPLAIN и EXPLAIN ANALYZE, Как читать, ANTI JOIN — нет ни одного совпадения, Hash Join, Materialize, Merge Join, Nested Loop Join, SEMI JOIN — есть ли хоть одно совпадение, Bitmap Heap Scan, BitmapAnd, BitmapOr, Exact vs Lossy bitmap, Index Only Scan, Index Scan + Filter, Index Scan, Sequence Scan + Filter, Sequence Scan

### Репликация
- `md_count`: 14, `flashcards_count`: 0
- Что разобрано: CDC и libslave, Eventual Consistency, Masterless (без ведущих узлов), Multi-Leader (Master-Master), Physical vs Logical репликация, Replication Lag и аномалии асинхронной репликации, SBR vs RBR (Statement vs Row Based Replication), Single-Leader (Master-Slave), Strong Consistency, Tunable Consistency (Cassandra), WAL (Write-Ahead Log), Репликация, Синхронная vs асинхронная репликация, топологии - сравнение

### Шардирование
- `md_count`: 9, `flashcards_count`: 0
- Что разобрано: Consistent Hashing, Virtual Buckets (виртуальное шардирование), Генерация ID в шардированных системах, Перебалансировка шардов (переналивка), Рандеву хеширование (Rendezvous, HRW), Роутинг запросов к шардам, Грабли шардирования, Стратегии роутинга шардов, Шардирование

### Партиционирование
- `md_count`: 9, `flashcards_count`: 0
- Что разобрано: pg partman, Вертикальное партиционирование, Партиционирование, Composite Partitioning, Hash Partitioning, List Partitioning, Range Partitioning, Термины партиционирования, Типы партиционирования

Coverage checklist: JOIN

| Пункт | Покрытие | Подтверждение |
|---|---|---|
| виды JOIN | есть | CROSS JOIN, FULL OUTER JOIN, INNER JOIN, LEFT JOIN |
| ON/WHERE | есть | ON vs WHERE в LEFT JOIN, Условие ON |
| NULL | есть | NULL в условиях JOIN |
| LATERAL | есть | LATERAL JOIN, LEFT LATERAL vs CROSS LATERAL |
| perf | есть | Производительность JOIN |

Coverage checklist: Индексы

| Пункт | Покрытие | Подтверждение |
|---|---|---|
| типы индексов (общая рамка) | есть | Индекс, Какие бывают индексы в PostgreSQL - сводка, Hash индекс |
| btree/brin/gin/gist/hash | есть | B-Tree vs LSM-Tree, B-tree индекс - внутреннее устройство, B-tree индекс - что это, GIN индекс (Generalized Inverted Index), GiST индекс (Generalized Search Tree) |
| composite/partial/covering | есть | Покрывающие индексы (covering index Index-Only Scan), Составные индексы (multi-column), Частичные индексы (partial index) |
| bloat/reindex | есть | REINDEX |

## Deep Dive: System Design (sd)

| Раздел | md_count | flashcards_count | кластеры |
|---|---:|---:|---|
| основные понятия | 69 | 0 | api-cards, backend-architecture-cards, balancing-cards, cache-patterns-cards, cache-strategies-cards, caching-basics-cards, eviction-algorithms-cards, observability-cards, performance-cards, properties-cards, proxy-cards |
| паттерны | 42 | 0 | architectural-patterns-cards, async-processing-cards, capacity-estimation-cards, client-server-communication-cards, deploy-strategies-cards, distributed-transactions-consensus-cards, event-driven-cards, fault-tolerance-cards, microservice-patterns-cards, mini-design-cards, requirements-cards |
| хранилища | 29 | 0 | db-choice-and-classes-cards, db-types-cards, index-cards |
| lesson5-cards | 22 | 0 | feed-design-detailed-cards, taxi-design-detailed-cards |
| lesson6-cards | 16 | 0 | — |
| распределенное хранение данных | 4 | 0 | cap |

Критичные кластеры (явное покрытие):

| Кластер | Покрытие | Подтверждение |
|---|---|---|
| балансировка | есть | DNS балансировка, L4 vs L7 балансировка, Random балансировка, Клиентская балансировка |
| кэширование | есть | Дизайн ленты - Feed Service + кэширование, Версионирование кэша, Тегированный кэш, Cache Ahead (опережающее кэширование) |
| отказоустойчивость | есть | Дизайн ленты - отказоустойчивость и геораспределённость, Дизайн такси - отказоустойчивость и масштабирование, Дизайн Booking - отказоустойчивость, масштабирование, сезонность, Дизайн Google Drive - отказоустойчивость и масштабирование |
| CAP/PACELC | есть | CAP — классификация БД, CAP-теорема — суть, PACELC — расширение CAP |
| Outbox/Inbox/Saga/2PC | есть | Дизайн Booking - бронирование через Saga, 2PC (Two-Phase Commit), 2PC vs Saga, Saga |
| capacity estimation | есть | Расчёт RPS (Requests Per Second), Расчёт нагрузки (Capacity Estimation) |

## Остальные домены Theory

| Домен | md_count | Кластеры (папки 1-го уровня) | Что разобрано (сэмпл basename) |
|---|---:|---|---|
| Rust | 62 | Какие-то вопросы, Сlosures, Смартпоинтеры | Borrowing (Заимствование), Box new(0u646 1000000000) - что произойдёт, Copy vs Clone, Enum, HashMap, Match, Mutex и RwLock - зачем в многопоточном коде, Orphan rule, Ownership (Владение), Send Sync ... (+2) |
| Брокеры сообщений | 39 | Apache Kafka, Push vs Pull, RabbitMQ, Общее | Архитектурные риски и Trade-offs, Без ключа (Key = null), Для чего и зачем, С ключом, Группы потребителей (Consumer Groups) и балансировка нагрузки, Механизм перебалансировки (Rebalancing), Параллелизм партиций как ключ к High Load, Под капотом, Сценарии масштабирования - Соотношение потребителей (C) и партиций (P), At least once (Минимум один раз) ... (+2) |
| Redis | 35 | Инвалидация кеша, Кеширование, Cash Miss, Масштабирование, Общее, Отказоустойчивость и Репликация (Sentinel), Падение Redis, Персистентность - RDB vs AOF, Проблемы кеширования, Проблемы распределенных систем - Split Brain и потери данных | Event-based (Delete on Write), Pub - Sub (через Redis, Kafka, NATS), TTL (Time-To-Live), Версионирование ключа, Инвалидация кэша, Зачем используют кеширование, Почему Redis быстрый (Performance), Что происходит ... (+5) |
| Синхронные интеграции | 30 | http | HTTP версии, HTTP что это, DELETE, PATCH - частичное изменение, POST - создание, PUT - полная замена, Методы синхронного взаимодействия, Сравнение HTTP методов, Level 0 - RPC поверх HTTP (НЕ REST), Level 1 - Ресурсы ... (+2) |
| Микросервисы и монолиты | 26 | Микросервис, Монолит | Stateful vs Stateless сервисы, Инфраструктурная и архитектурная сложность (Complexity), Ненадежность коммуникаций (Communication Issues), Согласованность данных (Data Consistency), Трудности отладки и мониторинга (Debugging & Tracing), Определение, Архитектура и механизм работы, Борьба с дубликатами (Идемпотентность), Зачем, Оркестрация (Orchestration) ... (+2) |
| grpc | 17 | основы, практика | REST vs gRPC, proto файл, Когда REST, когда gRPC + gRPC-Gateway, Номера полей и совместимость, Типы gRPC вызовов, Что такое gRPC, пример proto файла, Auth через interceptor, Deadlines и Timeouts ... (+3) |
| Docker | 9 | — | Docker Compose, Docker Container, Docker Image, Docker Volumes, Docker на Mac-Windows. Почему VM не вымерли, Dockerfile, Multi-stage build для Go, Слои Docker, Что это |
| архитектура кода | 9 | — | Clean Architecture в Go — практика, Clean Architecture, DDD — Aggregates, DDD — основные концепции, Hexagonal Architecture (Ports & Adapters), Hexagonal, Onion, Clean — одна идея, Layered Architecture (N-Layer), Onion Architecture ... (+1) |
| k8s | 3 | — | Зачем нужен Kubernetes, Что такое Kubernetes (k8s), Что такое Pod |
| курсы и доп материалы | 3 | — | hh.ru - PostgreSQL - Продвинутый, Без названия, Слитые курсы |

## Questions layer: покрытие по темам и компаниям

`questions/theory` — распределение по `topic`:

| topic | count |
|---|---:|
| Go | 40 |
| PostgreSQL | 26 |
| Architecture | 13 |
| OS | 9 |
| Kubernetes | 5 |
| Networking | 5 |
| Databases | 4 |
| subtopic: | 4 |
| Брокеры | 4 |
| DevOps | 3 |
| Algorithms | 2 |
| Redis | 2 |
| Security | 2 |
| Soft | 2 |
| System Design | 2 |
| Мониторинг | 2 |
| API | 1 |
| Development | 1 |
| Distributed Systems | 1 |
| Kafka | 1 |
| Networks | 1 |
| Rust | 1 |
| Testing | 1 |
| Архитектура | 1 |

`questions/tasks` — распределение по `topic`:

| topic | count |
|---|---:|
| Go | 80 |
| PostgreSQL | 16 |
| Algorithms | 10 |
| Architecture | 2 |
| System Design | 2 |

`questions/by-companies` — компании (файл существует):

- `avito.md`
- `cloudru.md`
- `cloudx.md`
- `employcity.md`
- `f6.md`
- `flant.md`
- `generics-с-trait-bounds.md`
- `groupib.md`
- `lamoda.md`
- `mediacom.md`
- `mts.md`
- `ozon.md`
- `pay2play.md`
- `realit.md`
- `stroki.md`
- `tradingview.md`
- `uzum.md`
- `vk.md`
- `wildberries.md`
- `x5.md`
- `xstack.md`
- `yandex.md`
- `амтех.md`
- `банк-точка.md`
- `бюро1400.md`
- `витех.md`
- `кск.md`
- `магнит.md`
- `мвидео.md`
- `олимпбет.md`
- `рсхб.md`
- `сбер.md`
- `ситидрайв.md`
- `техкон.md`

Нейминг-анализ topic (аномалии):
- `subtopic:` (4) — выглядит как технический мусор в frontmatter (`topic` заполнен некорректно).
- `Architecture` (13), `System Design` (2), `Distributed Systems` (1), `Архитектура` (1) — один домен разнесён по нескольким alias.
- `PostgreSQL` (26) и `Databases` (4) — пересечение/дублирование доменной метки.
- `Networking` (5) и `Networks` (1) — дубль по форме слова.
- `Брокеры` (4) и `Kafka` (1) — общий словарь топиков.

Связка theory ↔ questions (где глубина не совпадает):

| Домен theory | theory_md | questions/theory | questions/tasks | Оценка |
|---|---:|---:|---:|---|
| Go | 700 | 40 | 80 | покрытие относительно сбалансировано |
| sd | 182 | 16 | 4 | покрытие относительно сбалансировано |
| PostgreSQL | 116 | 26 | 16 | покрытие относительно сбалансировано |
| Rust | 62 | 1 | 0 | глубоко в theory, слабо в questions |
| Брокеры сообщений | 39 | 5 | 0 | покрытие относительно сбалансировано |
| Redis | 35 | 2 | 0 | покрытие относительно сбалансировано |
| Синхронные интеграции | 30 | 6 | 0 | покрытие относительно сбалансировано |
| Микросервисы и монолиты | 26 | 0 | 0 | покрытие относительно сбалансировано |
| grpc | 17 | 0 | 0 | покрытие относительно сбалансировано |
| Docker | 9 | 0 | 0 | покрытие относительно сбалансировано |
| архитектура кода | 9 | 0 | 0 | покрытие относительно сбалансировано |
| k8s | 3 | 0 | 0 | покрытие относительно сбалансировано |
| курсы и доп материалы | 3 | 0 | 0 | покрытие относительно сбалансировано |

## Качество базы: дубли, нейминг, рассинхроны

Дубли (по нормализованному slug, `questions/tasks`):
- `questions/tasks/001-min-cost-coupons.md` ; `questions/tasks/001_min_cost_coupons.md`
- `questions/tasks/025-ai-weather-high-load.md` ; `questions/tasks/025-ai-weather-highload.md`

Дубли (по нормализованному slug, `questions/theory`):
- Дубли не найдены.

Технический шум:
- `.DS_Store`: 35 шт.
  - `.DS_Store`
  - `theory/.DS_Store`
  - `questions/.DS_Store`
  - `theory/PostgreSQL/.DS_Store`
  - `theory/sd/.DS_Store`
  - `theory/Go/.DS_Store`
  - `theory/Go/Data Race vs Race Condition/.DS_Store`
  - `theory/Go/Каналы/.DS_Store`
  - `theory/Go/Lock-free и алгоритмы синхронизации/.DS_Store`
  - `theory/Go/runtime/.DS_Store`
  - `theory/Go/generics-reflect/.DS_Store`
  - `theory/Go/map/.DS_Store`
  - `theory/Go/structs/.DS_Store`
  - `theory/Go/unsafe/.DS_Store`
  - `theory/Go/slice-array-cards-v2/.DS_Store`
  - `theory/Go/memory-allocator/.DS_Store`
  - `theory/Go/functions/.DS_Store`
  - `theory/Go/errors/.DS_Store`
  - `theory/Go/sync/.DS_Store`
  - `theory/Go/context-iterators/.DS_Store`
  - `... (+15)`

Рекомендации по нормализации (без автоправок):
- Удалить/починить технический `topic: subtopic:` и ввести валидацию frontmatter.
- Свести алиасы System Design в единый набор (`Architecture`/`Архитектура`/`System Design`/`Distributed Systems`).
- Свести `PostgreSQL` и `Databases` к одному правилу topic-нейминга.
- Свести `Networking` и `Networks` к одному topic.
- Для `questions/tasks` зафиксировать единый slug-формат (`-` или `_`) и уникальность на уровне CI-скрипта.
- Периодически чистить `.DS_Store` в `main/interviews` и подкаталогах.

## Что обновлять руками при добавлении новых тем

Чеклист ручного апдейта:
1. Добавить/обновить карточки в `theory/...` и убедиться, что имена отражают подтему.
2. Для Go-подтем проверить `flashcards_count` и `flashcards_coverage`.
3. Для PostgreSQL явно обновить блоки `JOIN`, `Индексы`, `Транзакции`, `Оптимизация`, `Репликация`, `Шардирование`, `Партиционирование`.
4. Для `sd` обновить кластеры по папкам `основные понятия/паттерны/хранилища/lesson5/lesson6/распределенное хранение`.
5. Проверить распределение `questions/theory` и `questions/tasks` по `topic` и нейминг-аномалии.
6. Обновить список `questions/by-companies` при добавлении новой компании.

Мини-набор команд для пересчёта метрик:
```bash
cd /Users/caoguojun0/Documents/Obsidian/main/interviews
find . -type f -name '*.md' | wc -l
find theory -type f -name '*.md' | awk -F/ '{print $2}' | sort | uniq -c | sort -nr
find theory/Go -type f -name '*.md' | rg -v '/flashcards/' | wc -l
find theory/Go -type f -name '*.md' | rg '/flashcards/' | wc -l
find questions/theory -type f -name '*.md' | wc -l
find questions/tasks -type f -name '*.md' | wc -l
```

Правило отражения новой подтемы:
- Если новая тема в `theory/Go` -> обновлять таблицу Go + соответствующий обязательный блок.
- Если новая тема в `theory/PostgreSQL` -> обновлять таблицу PostgreSQL + coverage checklist `JOIN/Индексы` (если релевантно).
- Если новая тема в `theory/sd` -> обновлять таблицу SD + критичные кластеры.
- Если новая тема добавлена только в `questions` -> обновлять раздел `Questions layer` и оценку рассинхрона.
