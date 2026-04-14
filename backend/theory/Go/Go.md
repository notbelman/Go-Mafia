## Быстрая навигация

- [[Версии Go]] — ключевые изменения Go 1.21–1.26 (changelog на русском)
- [[Structs]] — структуры, embedding, выравнивание, паттерны
- [[Interfaces]] — iface/eface, type assertion, nil interface
- [[Функции]] — closures, defer, calling conventions, inlining
- [[Errors]] — error interface, паника, wrapping, Is/As
- [[Slices]] — внутреннее устройство, операции, ловушки
- [[backend/theory/Go/Map/Map]] — хэш-мапа под капотом, операции
- [[Strings]] — string/[]byte/rune, юникод, оптимизация
- [[Defer]] — механика, LIFO, взаимодействие с panic/recover
- [[Unsafe]] — unsafe.Pointer, uintptr, правила использования
- [[backend/theory/Go/Каналы/Каналы]] — hchan, паттерны, select, буфер, nil-каналы
- [[backend/theory/Go/Sync/Sync]] — Mutex, RWMutex, atomic, sync.Map, паттерны
- [[Context]] — WithCancel/Timeout/Value, errgroup, правила
- [[Runtime]] — GMP scheduler, GC, preemption
- [[Memory]] — стек vs куча, аллокатор, escape analysis
- [[Конкурентность — глоссарий]] — термины: deadlock, race, CAS, happens-before
- [[Lock-free и алгоритмы синхронизации]] — lock-free DS, Treiber stack, Michael-Scott queue
- [[Generics]] — constraints, type inference, mixin, trade-offs
- [[Reflect]] — рефлексия, три свойства, теги структур
- [[Iterators]] — итераторы в Go 1.22+
- [[Архитектура]] — Clean Arch, DDD, Hexagonal, Onion
- [[ООП]] — SOLID, композиция vs наследование, полиморфизм

---

## Основы языка

### [[Structs]]
- [[Структуры основы]]
- [[Методы и ресиверы]] / [[Value vs Pointer receiver]]
- [[Встраивание типов]] / [[Встраивание когда и когда нет]]
- [[Выравнивание структур]] / [[Выравнивание инструменты и практика]]
- [[Functional Options]] / [[Type alias vs Type definition]]
- [[Пустые структуры]] / [[DOD Data Oriented Design]]

### [[Interfaces]]
- [[Что такое интерфейс]] / [[iface структура]] / [[eface]]
- [[nil интерфейса]] / [[Кэш itab]]
- [[Type assertion и type switch]] / [[Стоимость type assertion]]
- [[Дженерики vs Интерфейсы]] / [[Мономорфизация]]
- [[Best practices]] / [[Расположение интерфейсов]]

### [[Функции]]
- [[Функции основы]] / [[Функции первого класса]]
- [[Calling conventions и Go ABI]] / [[Inlining функций]]
- [[Замыкание]] / [[Именованные возвращаемые значения]]
- [[Декоратор и композиция]] / [[Каррирование и ленивые вычисления]]

### [[Errors]]
- [[Интерфейс error]] / [[Sentinel ошибки]]
- [[Оборачивание ошибок]] / [[errors.Is и errors.As]]
- [[Паника и recover]] / [[Defer и паники]]
- [[Multierror]] / [[Стектрейс ошибки]]

### [[Defer]]
- [[defer под капотом]]
- [[основы]]
- [[продвинутое]]
- [[Почему Mutex.Unlock() лучше не в defer]]

### [[Unsafe]]
- [[unsafe.Pointer и uintptr]]
- [[Sizeof,Alignof,Offsetof]]

---

## Типы данных

### [[Slices]]
- Внутреннее устройство (len/cap/ptr), growth алгоритм
- Ловушки при append, slice tricks, copy

### [[backend/theory/Go/Map/Map]]
- Хэш-таблица под капотом, buckets, overflow
- Конкурентный доступ, map + RWMutex vs sync.Map

### [[Strings]]
- string как []byte, rune vs byte, utf-8
- strings.Builder, оптимизация конкатенации

---

## Конкурентность

### [[backend/theory/Go/Каналы/Каналы]]
- [[Что это]] / [[Поля структуры hchan]]
- [[Буферизированные каналы]] / [[Небуферизированные каналы]]
- [[Deadlock в каналах]] / [[Nil-канал]]
- [[select]] / [[Паттерны]]

### [[backend/theory/Go/Sync/Sync]]
- [[sync.Mutex/Структура|sync.Mutex]] / [[sync.RWMutex]] / [[sync - atomic]]
- [[sync.WaitGroup]] / [[sync.Once]] / [[sync.Pool]] / [[sync.Cond]]
- [[sync.Map]] / [[Паттерны]]

### [[Context]]
- [[Context.md]] / [[Context ошибки и правила]]
- [[context WithCancel]] / [[context WithTimeout]] / [[context WithDeadline]]
- [[context WithValue]] / [[context WithoutCancel]] / [[context AfterFunc]]
- [[errgroup с контекстами]] / [[Оборачивание функций без контекста]]

### [[Конкурентность — глоссарий]]
Atomicity, CAS, Data Race vs Race Condition, Deadlock, Livelock, Starvation, Happens-Before, Spinlock, Semaphore

### [[Lock-free и алгоритмы синхронизации]]
ABA-проблема, Treiber Stack, Michael-Scott Queue, RCU, Актор модель, Закон Амдала

---

## Рантайм

### [[Runtime]]
- **Scheduler**: [[GMP обзор]], [[work_stealing]], [[preemption]], [[netpoller]], [[sysmon]]
- **GC**: [[Обзор GC]], [[Tri-color marking]], [[Write barrier]], [[GC Pacer]], [[GOGC и GOMEMLIMIT]], [[Green Tea GC (new)]], [[Green Tea - Vector acceleration (new)]]

### [[Memory]]
- **Стек**: [[Стек vs Куча]], [[Stack growth]], [[Стек]]
- **Аллокатор**: [[Три уровня аллокатора (mcache - mcentral - mheap)]], [[Классы размеров (size classes)]]
- **Escape**: [[Escape analysis - что это и зачем]], [[Что вызывает escape]]
- **Практика**: [[Практические приёмы уменьшения аллокаций]]

---

## Дополнительно

### [[Generics]]
- [[Зачем дженерики]] / [[Constraints]] / [[Type inference и параметры типов]]
- [[Когда использовать и цена дженериков]] / [[Ограничения дженериков]]
- [[Mixin и CRTP]] / [[Обобщённые структуры и type definitions]]

### [[Reflect]]
- [[Три свойства рефлексии]] / [[Возможности рефлексии]]
- [[Практические кейсы рефлексии]] / [[Теги структур]]
- [[Недостатки рефлексии]] / [[Рефлексия vs интроспекция]]

### [[Iterators]]
- [[Итераторы]] / [[Итераторы применение]] / [[Итераторы горутины и ошибки]]

### [[Архитектура]]
- Clean Architecture, DDD, Hexagonal (Ports & Adapters), Onion, Layered

### [[ООП]]
- SOLID (S/O/L/I/D), Композиция vs Наследование, Полиморфизм, Энкапсуляция
