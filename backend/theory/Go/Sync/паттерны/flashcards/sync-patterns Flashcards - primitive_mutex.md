#flashcards/sync-patterns/primitive_mutex

Что такое примитивный мьютекс? Чем он принципиально отличается от spin lock?
?
![[Примитивный_мьютекс#^primitive-spinning-problem]]

Что такое park и unpark? Через что они реализованы в Linux?
?
![[Примитивный_мьютекс#^futex-wait]]
![[Примитивный_мьютекс#^futex-wake]]

Опиши fast path мьютекса. Сколько системных вызовов он делает?
?
![[Примитивный_мьютекс#^primitive-fast-path]]

Опиши slow path мьютекса. Почему он не жжёт CPU в отличие от spin lock?
?
![[Примитивный_мьютекс#^primitive-slow-path]]

Что такое futex? Как работает его быстрый путь?
?
![[Примитивный_мьютекс#^futex-fast-path]]

Как работает futex_wait? При каком условии поток засыпает?
?
![[Примитивный_мьютекс#^futex-wait]]

Опиши гибридный подход в реальных мьютексах (spin + park).
?
![[Примитивный_мьютекс#^primitive-hybrid]]

Опиши алгоритм Lock() в Go sync.Mutex по шагам.
?
![[Примитивный_мьютекс#^go-mutex-algorithm]]

Что такое Normal и Starvation режимы Go sync.Mutex? Когда переключается?
?
![[Примитивный_мьютекс#^go-mutex-normal-mode]]
![[Примитивный_мьютекс#^go-mutex-starvation-mode]]

Через сколько миллисекунд ожидания Go sync.Mutex переключается в starvation режим?
?
![[Примитивный_мьютекс#^go-mutex-starvation-mode]]

Что выведет этот код?
```go
var mu sync.Mutex
var results []int

for i := 0; i < 5; i++ {
    i := i
    go func() {
        mu.Lock()
        results = append(results, i)
        mu.Unlock()
    }()
}

time.Sleep(100 * time.Millisecond)
sort.Ints(results)
fmt.Println(results)
```
?
`[0 1 2 3 4]` — мьютекс корректно защищает append, все 5 элементов добавляются. Порядок в slice неопределён (зависит от планировщика), поэтому сортируем перед выводом. Без мьютекса — гонка на append.
![[Примитивный_мьютекс#^primitive-impl]]
