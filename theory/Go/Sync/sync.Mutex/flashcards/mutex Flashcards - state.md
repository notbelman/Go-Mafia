#flashcards/mutex/state

Как устроено поле state в sync.Mutex? Сколько бит и что в них хранится?
?
![[state#^state-layout]]

Какой бит отвечает за Locked, Woken, Starving в state? Какие числовые значения масок?
?
![[state#^state-bits]]

Где в state хранится счётчик ожидающих горутин и сколько бит под него отведено?
?
![[state#^state-bit-split]]

state = 9. Что это означает? Раскодируй вручную.
?
![[state#^state-examples]]

state = 21. Что это означает?
?
![[state#^state-examples]]

Как атомарно прочитать количество ожидающих горутин из state?
?
![[state#^state-read]]

Как атомарно установить флаг Locked в state? Как сбросить?
?
![[state#^state-write]]

Как атомарно увеличить счётчик waiters в state на 1?
?
![[state#^state-write]]

Что выведет этот код?
```go
var mu sync.Mutex
mu.Lock()
// state после Lock?
s := reflect.ValueOf(&mu).Elem().Field(0).Int()
fmt.Println(s & 1)   // locked?
fmt.Println(s >> 3)  // waiters?
```
?
`1` и `0` — после Lock() без конкуренции: бит Locked = 1, waiters = 0 (никто не ждёт).
![[state#^state-read]]
