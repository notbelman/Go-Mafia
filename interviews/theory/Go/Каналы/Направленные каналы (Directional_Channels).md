## Что это

Ограничение канала на только отправку или только получение: ^directional-def
```go
chan<- int   // только отправка (send-only)
<-chan int   // только получение (receive-only)
chan int     // двунаправленный (обычный)
```
^directional-syntax

```go
func producer(out chan<- int) {
    out <- 42    // ✅ можно отправлять
    val := <-out // ❌ compile error: cannot receive from send-only channel
}

func consumer(in <-chan int) {
    val := <-in  // ✅ можно получать
    in <- 42     // ❌ compile error: cannot send to receive-only channel
}

func main() {
    ch := make(chan int) // обычный канал
    go producer(ch)      // автоматически сужается до chan<-
    go consumer(ch)      // автоматически сужается до <-chan
}
```
^directional-example

## Под капотом — один и тот же канал

Направленность — это **ограничение компилятора**, не рантайма. Под капотом это тот же `runtime.hchan`. Никакой разницы в структуре, производительности или поведении нет. Компилятор просто не даёт вызвать запрещённую операцию. ^directional-compiler-only

```go
ch := make(chan int)

var sendOnly chan<- int = ch  // ок, сужение типа
var recvOnly <-chan int = ch  // ок, сужение типа

// Обратно расширить нельзя:
// var full chan int = sendOnly  // ❌ compile error
```
^directional-narrowing

**Сужение типа — односторонний процесс:** `chan int` → `chan<- int` или `<-chan int` — можно. Обратно расширить нельзя. ^directional-one-way

## Зачем

Контракт на уровне типов: функция явно говорит "я только пишу" или "я только читаю". Ошибку ловишь на этапе компиляции, а не в рантайме. ^directional-why
