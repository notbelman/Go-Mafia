#flashcards/sync-primitive-supple/mutex_mistakes

Какой принцип гранулярности мьютекса? В чём суть аналогии с кассой?
?
![[Практические_ошибки_с_мьютексами#^mutex-granularity-principle]]

Почему мьютекс над локальной переменной бессмысленен? Что выведет код?
```go
go func() {
    var value int
    mu.Lock()
    for i := 0; i < 10; i++ { value++ }
    mu.Unlock()
    fmt.Println(value)
}()
```
?
Выведет `10`. Мьютекс бессмысленный overhead — `value` локальная, её никто не шарит между горутинами.
![[Практические_ошибки_с_мьютексами#^mutex-unnecessary-sync]]

Что произойдёт если каждая горутина создаёт свой локальный sync.Mutex?
?
![[Практические_ошибки_с_мьютексами#^mutex-must-be-shared]]

Что не так в этом коде с точки зрения гранулярности?
```go
mu.Lock()
item := cache[key]
fmt.Println(item)
mu.Unlock()
```
?
`fmt.Println` выполняется под мьютексом — это лишний overhead и блокировка других горутин. Нужно вынести вывод за пределы критической секции.
![[Практические_ошибки_с_мьютексами#^mutex-granularity-code]]

Почему возврат среза из-под мьютекса — data race? Как исправить?
?
![[Практические_ошибки_с_мьютексами#^mutex-ref-leak-code]]

Что выведет этот код? Есть ли здесь гонка?
```go
func (b *Buffer) GetAll() []int {
    b.mu.Lock()
    defer b.mu.Unlock()
    return b.data
}
// Где-то снаружи:
items := buf.GetAll()
items[0] = 999  // пишем без блокировки
```
?
Поведение гонки — data race. `items` указывает на тот же underlying array что и `b.data`. Запись без мьютекса = гонка с другими горутинами читающими/пишущими через методы Buffer.
![[Практические_ошибки_с_мьютексами#^mutex-ref-leak-code]]

Что такое проблема API в контексте top() + pop()? Почему синхронизации каждого метода недостаточно?
?
![[Практические_ошибки_с_мьютексами#^mutex-api-toctou]]

Что произойдёт в этом коде?
```go
func (c *Cache) Get(key string) int {
    c.mu.Lock()
    defer c.mu.Unlock()
    if c.Size() > 0 { return c.data[key] }
    return 0
}
func (c *Cache) Size() int {
    c.mu.Lock()
    defer c.mu.Unlock()
    return len(c.data)
}
```
?
Deadlock — `Get` захватывает мьютекс, затем вызывает `Size()`, который пытается захватить тот же мьютекс. Go мьютекс не reentrant.
![[Практические_ошибки_с_мьютексами#^mutex-recursive-locked-pattern]]

Что означает суффикс `Locked` в приватных методах? Из какой библиотеки этот паттерн?
?
![[Практические_ошибки_с_мьютексами#^mutex-recursive-locked-pattern]]

Почему плохо встраивать sync.Mutex в структуру? Что это нарушает?
?
![[Практические_ошибки_с_мьютексами#^mutex-embed-problem]]
