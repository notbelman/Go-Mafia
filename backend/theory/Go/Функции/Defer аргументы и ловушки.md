- аргументы defer-функции вычисляются **сразу** при откладывании (копия), не при выходе ^da-eager-eval
- defer nil-функции → **panic** (nil pointer dereference) ^da-nil-panic
- переприсвоение функции **после** defer не влияет — отложена старая ^da-reassign
- fix: замыкание (захватить переменную по ссылке) или указатель ^da-fix

---
[[functions Flashcards - defer_args]]
## Аргументы вычисляются сразу

```go
func process() {
    status := ""
    defer notify(status)  // status="" скопирован СЕЙЧАС
    status = "error"      // поздно — defer уже запомнил ""
}
func notify(s string) { fmt.Println(s) }
// Вывод: "" (пустая строка)
```

**Почему:** `runtime.deferproc` вызывается в момент defer → аргументы нужны сейчас → копия. ^da-why-eager

## Как починить

```go
// Способ 1: замыкание (захватывает переменную, не значение)
defer func() { notify(status) }()  // status прочитается при ВЫХОДЕ

// Способ 2: указатель
defer notifyPtr(&status)  // копируется адрес, а не значение
```

^da-fix-closure

## Пример из лекции: 1-2-3

```go
func process() {
    defer handle(get())  // get() вызывается СРАЗУ → печатает "1"
    fmt.Println("2")     // печатает "2"
    // выход → handle() выполняется → печатает "3"
}
// Вывод: 1 2 3
```

Аргумент `get()` для defer вычисляется немедленно, хотя `handle()` отложена. ^da-123-example

## Defer nil-функции

```go
var f func()           // f = nil
defer f()              // отложили nil
f = func() { ... }    // переприсвоили — но defer уже запомнил nil
// при выходе: panic: runtime error: invalid memory address
```

Переприсвоение **после** defer не влияет. Отложено то, что было в момент defer. ^da-nil-detail

## Defer в цикле — ловушка

```go
for _, path := range paths {
    f, _ := os.Open(path)
    defer f.Close()  // ⚠️ все Close вызовутся только при выходе из ФУНКЦИИ
    // файлы копятся открытыми → утечка ресурсов
}

// Fix: обернуть в функцию
for _, path := range paths {
    func() {
        f, _ := os.Open(path)
        defer f.Close()  // ✅ Close при выходе из анонимной функции
        // ...
    }()
}
```

^da-loop-trap

## Связь
- [[Defer механика и порядок]] — LIFO, привязка к функции
- [[Defer и именованные возвращаемые]] — модификация результата
- [[Функции основы]] — всё копируется
