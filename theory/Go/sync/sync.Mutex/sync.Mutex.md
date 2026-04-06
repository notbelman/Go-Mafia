## sync.Mutex

#### [[Структура]]
Поля: state (int32) + sema (uint32)

#### [[state]]
Биты: Locked(0) | Woken(1) | Starving(2) | WaiterShift(3+)

#### [[sema (семафор)]]
Runtime semaphore: park ожидающих, goready при Unlock

#### [[Lock()]]
Fast path: CAS(0→1). Slow path: спин → gopark. Starvation mode

#### [[Unlock()]]
Снять Locked бит, разбудить следующего ожидающего

#### [[Два режима (с Go 1.9)]]
Normal: FIFO нарушается в пользу spinning. Starving: строгий FIFO

#### [[sync.Mutex.TryLock (Go 1.18+)]]
Non-blocking попытка захвата — возвращает false если занят

#### [[пример]]
Типичные паттерны использования Mutex
