## Sync в Go

### Примитивы

#### [[sync.Mutex vs sync.RWMutex]]
Когда Mutex, когда RWMutex — reader/writer contention

#### [[sync.WaitGroup]]
WaitGroup, Add/Done/Wait, типичные ошибки

#### [[sync.Wg пример]]
Практический пример использования WaitGroup

#### [[sync.Once]]
Ленивая инициализация, once.Do, singleton паттерн

#### [[sync.Pool]]
Пул объектов, GC и Pool, уменьшение аллокаций

#### [[sync.Cond]]
Условная переменная, Wait/Signal/Broadcast, spurious wakeup

#### [[Runtime internals]]
Внутреннее устройство синхронизационных примитивов в рантайме

#### [[Go Memory Model и Memory Barriers]]
Happens-before, memory barriers, sync как fence

---

## [[sync.Mutex]]

#### [[Структура]]
Поля state и sema, битовые флаги

#### [[state]]
Locked/Woken/Starving биты, waiter count

#### [[sema (семафор)]]
Runtime semaphore, gopark/goready, очередь ожидающих

#### [[Lock()]]
Fast path (CAS), slow path (спин + сон), starvation mode

#### [[Unlock()]]
Освобождение, пробуждение ожидающего, передача владения

#### [[sync.Mutex.TryLock (Go 1.18+)]]
Non-blocking попытка захвата

#### [[Два режима (с Go 1.9)]]
Normal mode vs starvation mode — честность vs производительность

#### [[пример]]
Практический пример с Mutex

---

## [[sync.RWMutex]]

#### [[sync.RWMutex]]
Структура, readerCount, readerWait

#### [[RLock()]]
Захват read lock, fast path vs slow path

#### [[RUnlock()]]
Освобождение read lock, пробуждение writers

#### [[Lock()]]
Захват write lock, ожидание всех readers

#### [[Unlock()]]
Освобождение write lock

#### [[TryRLock TryLock (Go 1.18+)]]
Non-blocking варианты

#### [[пример]]
Практический пример

---

## [[sync.Map]]

#### [[sync.Map]]
Зачем нужен, когда лучше map + RWMutex

#### [[sync.Map vs map + RWMutex — Когда Что Использовать]]
Сравнительная таблица, сценарии использования

#### [[Структура]]
read map + dirty map + mu + misses

#### [[Cтруктура]]
Внутренняя структура read/dirty

#### [[Load()]]
Fast path (read map), slow path (dirty map)

#### [[Store()]]
Запись, promoted vs dirty path

#### [[Delete()]]
Мягкое удаление через expunged pointer

#### [[Range]]
Итерация, promotion dirty → read

#### [[amended]]
Флаг "dirty содержит новые ключи"

#### [[Promotion]]
Продвижение dirty → read map, сброс misses

---

## sync - atomic

#### [[sync - atomic]]
Пакет atomic: Load/Store/Add/CAS/Swap

#### [[atomic.Value]]
Атомарное хранение произвольного значения, Store/Load/CompareAndSwap

#### [[Atomic vs Mutex]]
Когда atomic быстрее, когда Mutex нужен

#### [[Memory Ordering]]
Порядок памяти, happens-before в atomic операциях

#### [[Почему НЕ atomic везде]]
Сложность reasoning, ABA, не для составных операций

---

## Паттерны

#### [[Spin lock]]
Busy-wait локи, когда оправданы (очень короткие критические секции)

#### [[Примитивный мьютекс]]
Мьютекс через atomic CAS

#### [[Семафор]]
Счётный семафор через channel или atomic

#### [[Recursive и Timed mutex]]
Рекурсивный мьютекс (антипаттерн), тimed lock

#### [[Ticket lock]]
FIFO-честный лок через счётчик

#### [[CAS паттерны]]
Compare-and-swap идиомы: lock-free update

#### [[True sharing и False sharing]]
Кэш-линии, false sharing и как избежать (padding)

#### [[Spurious wakeup]]
Ложные пробуждения в sync.Cond, always loop

#### [[Мьютекс Петерсона]]
Алгоритм Петерсона — программный мьютекс без атомарных инструкций
