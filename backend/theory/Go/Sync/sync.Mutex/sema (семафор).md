Механизм усыпления и пробуждения горутин. ^sema-purpose

Спящая горутина не ест CPU. Без семафора пришлось бы крутиться в цикле (busy wait). ^sema-why

```go
sema uint32  // счётчик + очередь спящих в runtime
```
^sema-field

## Две функции

| Функция                          | Что делает                  | Используется |
| :------------------------------- | :-------------------------- | ------------ |
| `runtime_SemacquireMutex(&sema)` | уснуть, встать в очередь    | Lock()       |
| `runtime_Semrelease(&sema)`      | разбудить одного из очереди | Unlock()     |

^sema-functions

## Как работает
```
Lock() не смог захватить:
    state: waiters++
    runtime_SemacquireMutex(&sema)  --> горутина спит

Unlock():
    state: locked = 0
    если waiters > 0:
        runtime_Semrelease(&sema)   --> будит одного
```
^sema-flow

## Связь
- [[sync.Mutex/Структура]] — поля mutex
- [[sync.Mutex/Lock()]] — runtime_SemacquireMutex в slow path
- [[sync.Mutex/Unlock()]] — runtime_Semrelease
