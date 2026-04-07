#flashcards/sync-primitive-supple/cas

Что делает CAS (CompareAndSwap)? Опиши семантику операции и сколько CPU инструкций это занимает.
?
![[CAS_и_CAS_loop#^cas-definition]]

Какую CPU инструкцию использует CAS на x86 и почему между проверкой и записью никто не может вклиниться?
?
![[CAS_и_CAS_loop#^cas-cpu-instruction]]

Почему Load + Store раздельно — это race condition? Покажи проблему на примере.
?
![[CAS_и_CAS_loop#^cas-load-store-race]]

Что выведет этот код при запуске двух горутин одновременно?
```go
var initialized atomic.Bool
var m map[string]int

func init() {
    if !initialized.Load() {
        initialized.Store(true)
        m = make(map[string]int)
    }
}
```
?
Поведение непредсказуемо — обе горутины могут прочитать false одновременно, обе выполнят инициализацию, map создастся дважды. Это race condition между Load и Store.
![[CAS_и_CAS_loop#^cas-load-store-race]]

Опиши паттерн CAS loop по шагам. Что происходит когда CAS не проходит?
?
![[CAS_и_CAS_loop#^cas-loop-concept]]

Что выведет этот код? Есть ли здесь race condition?
```go
var counter int64

func main() {
    var wg sync.WaitGroup
    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            for {
                cur := atomic.LoadInt64(&counter)
                if atomic.CompareAndSwapInt64(&counter, cur, cur+1) {
                    return
                }
            }
        }()
    }
    wg.Wait()
    fmt.Println(counter)
}
```
?
`1000` — CAS loop гарантирует корректный атомарный инкремент без race condition. Если CAS не прошёл (кто-то изменил между Load и CAS), горутина повторяет с новым значением.
![[CAS_и_CAS_loop#^cas-loop-code]]

На каких lock-free структурах данных основан CAS loop? Какая у них гарантия прогресса?
?
![[CAS_и_CAS_loop#^cas-lock-free-structures]] + ![[CAS_и_CAS_loop#^cas-progress-guarantee]]

Когда CAS loop избыточен и что использовать вместо него?
?
![[CAS_и_CAS_loop#^cas-not-needed]]

Что выведет этот код?
```go
var counter int64

func main() {
    value := atomic.AddInt64(&counter, 1)
    if value%100 == 0 {
        fmt.Println("checkpoint")
    }
    fmt.Println(value)
}
```
?
`1` (и "checkpoint" не выводится). `atomic.AddInt64` возвращает новое значение атомарно — можно проверять его локально без дополнительного CAS.
![[CAS_и_CAS_loop#^cas-not-needed]]

Почему sync.Once внутри использует mutex, а не CAS?
?
Чтобы вторая горутина дождалась завершения f() — CAS не блокирует ожидающего, а мьютекс заставляет подождать.
![[CAS_и_CAS_loop#^cas-not-needed]]
