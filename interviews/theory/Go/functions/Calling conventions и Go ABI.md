- **calling convention** — правила вызова: как передаются аргументы и кто чистит стек ^cc-definition
- **stack-based** — аргументы через стек (push/pop); **register-based** — через регистры (быстрее) ^cc-types
- Go до 1.17 (amd64): **stack-based**. Go 1.17+: **register-based ABI** (~40% быстрее доступ к аргументам) ^cc-go-history
- регистров ограниченное количество → большое число аргументов всё равно идёт через стек ^cc-register-limit
- stack overflow в Go: ~1 ГБ (64-бит) / 512 МБ (32-бит); **нельзя** перехватить recover ^cc-stackoverflow

---

## Stack-based vs Register-based

```
Stack-based (до Go 1.17):
  push arg2       ; аргумент в стек
  push arg1       ; аргумент в стек
  call func       ; вызов
  ; внутри func: pop со стека → в регистр → операция

Register-based (Go 1.17+ amd64):
  mov RAX, arg1   ; аргумент сразу в регистр
  mov RBX, arg2   ; аргумент сразу в регистр
  call func       ; вызов
  ; внутри func: аргументы уже в регистрах → операция
```

Разница: stack-based → push в память + pop из памяти. Register-based → данные уже в регистрах, не нужен поход в RAM. ^cc-diff-mechanism

## Бенчмарк

На искусственных тестах доступ к аргументам в регистрах **~40% быстрее**, чем на стеке (Go 1.17, amd64). ^cc-benchmark

## Кто чистит стек

| Convention | Аргументы | Кто чистит стек |
| :--------- | :-------- | :-------------- |
| stdcall    | стек (обратный порядок) | вызываемая (ret N) |
| cdecl      | стек (обратный порядок) | вызывающая (add SP, N) |
| fastcall   | первые 2 в регистрах, остальные на стеке | вызываемая |

Go использует свой calling convention, не совпадающий с C. ^cc-who-cleans

## Ограничение регистров

```go
// Мало аргументов — всё в регистрах (быстро)
func add(a, b int) int { return a + b }

// Много аргументов — часть на стеке (регистров не хватит)
func mega(a, b, c, d, e, f, g, h, i, j int) int { ... }
```

^cc-register-spill

## Stack overflow

```go
func infinite() {
    var buf [10_000_000]byte  // 10 МБ на стеке — быстро переполним
    _ = buf
    infinite()
}

func main() {
    defer func() {
        recover()  // ❌ НЕ поможет — stack overflow нельзя перехватить
    }()
    infinite()
    // runtime: goroutine stack exceeds 1000000000-byte limit
    // fatal error: stack overflow
}
```

Stack overflow — **fatal error**, не panic. Recover не работает. Макс. стек горутины: ~1 ГБ (64-бит). ^cc-stackoverflow-fatal

## Связь
- [[Аппаратный стек и вызов функций]] — SP, BP, IP, CALL/RET
- [[Inlining функций]] — inlining убирает overhead calling convention
- [[Функции основы]] — всё копируется при передаче
