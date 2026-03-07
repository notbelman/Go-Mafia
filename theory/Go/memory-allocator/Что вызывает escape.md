- принцип: компилятор не может доказать, что переменная не переживёт scope → куча
- основные триггеры: return указателя, closure, interface boxing, канал, глобальная переменная, слайс, слишком большой объект, запись указателя в map/slice
- `make([]int, n)` с runtime-размером → куча; `make([]int, 3)` с константой → стек

---

[[memory Flashcards - escape_triggers]]

Принцип: если компилятор не может доказать что переменная не переживёт свой scope — она уходит на кучу. ^esc-principle

```go
// Указатель уходит из функции
func f() *int {
    x := 42
    return &x              // escape: указатель переживёт стек функции
}
// Return указателя на локальную переменную → escape: указатель 
// переживёт стек функции. 
```
^esc-return-pointer

```go
// Closure захватывает переменную
func f() {
    x := 42
    go func() {
        fmt.Println(x)     // escape: горутина может жить дольше f()
    }()
}
//Closure (особенно в горутине) захватывает переменную → escape: 
//горутина может жить дольше f(). 
```
^esc-closure

```go
// Interface boxing
var i interface{} = 42     // escape: компилятор не знает размер за интерфейсом
fmt.Println(x)             // то же самое: аргументы упаковываются в interface{}
                           // любой вызов fmt.Print/Println/Sprintf → куча
```

^a8e850

Interface boxing → escape: компилятор не знает тип/размер за интерфейсом. Любой вызов `fmt.Print/Println/Sprintf` → куча. ^esc-interface

```go
// Слайс vs массив
s := []int{1, 2, 3}       // escape: слайс содержит указатель на underlying array
a := [3]int{1, 2, 3}      // НЕ escape: массив — value type, копируется целиком
```

^83d48e

Слайс → escape (содержит указатель на underlying array). Массив фиксированного размера → НЕ escape (value type, копируется). ^esc-slice-vs-array

```go
// Слишком большой объект
var a [10_000_000]int      // escape: не влезает в стек, даже без return &a
```
Слишком большой объект → escape: не влезает в стек даже без return. ^esc-too-large

```go
// Передача в канал
ch <- &x                   // escape: компилятор не знает кто и когда прочитает
```
Передача указателя в канал → escape: компилятор не знает кто и когда прочитает. ^esc-channel

```go
// make с неизвестным размером
s := make([]int, n)        // escape: n известен только в runtime
s := make([]int, 3)        // НЕ escape: размер известен компилятору
```
`make([]int, n)` с runtime-размером → escape. `make([]int, 3)` с константой → НЕ escape. ^esc-make-size

```go
// Запись в глобальную переменную
var global *int
func f() {
    x := 1
    global = &x            // escape: переживает scope функции
}
```
Запись указателя в глобальную переменную → escape: переживает scope функции. ^esc-global

```go
// Запись указателя в map/slice
m[key] = &x               // escape: компилятор не может отследить lifetime
slice = append(slice, &x)  // escape: то же самое
```
Запись указателя в map или slice → escape: компилятор не может отследить lifetime. ^esc-map-slice

## Связь
- [[Escape analysis - что это и зачем]] — что такое escape analysis и зачем нужен
- [[Inlining и escape]] — inlining может отменить escape
- [[Практические приёмы уменьшения аллокаций]] — как писать код с меньшим escape
