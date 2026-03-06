#flashcards/errors/recoverable

Почему деление целого числа на ноль в Go вызывает панику, а не UB?
?
![[Восстановимые_и_невосстановимые_ошибки#^go-safe-div]]

Что произойдёт при делении `1.0 / 0.0` в Go? Почему нет паники?
?
![[Восстановимые_и_невосстановимые_ошибки#^float-div-zero]]

Что произойдёт при разыменовании nil pointer? Можно ли recover?
?
![[Восстановимые_и_невосстановимые_ошибки#^nil-deref]]

Что произойдёт при stack overflow? Почему recover не помогает?
?
![[Восстановимые_и_невосстановимые_ошибки#^stack-overflow]]

Что произойдёт при OOM? Почему recover не помогает?
?
![[Восстановимые_и_невосстановимые_ошибки#^oom]]

Заполни таблицу: для каждой ситуации — паника или нет, recover или нет: int/0, float/0, nil deref, stack overflow, OOM.
?
![[Восстановимые_и_невосстановимые_ошибки#^recover-table]]

Что выведет этот код?
```go
func main() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("recovered:", r)
        }
    }()
    x := 1 / 0
    _ = x
}
```
?
`recovered: runtime error: integer divide by zero` — деление int на ноль вызывает recoverable панику. defer с recover перехватывает её.
![[Восстановимые_и_невосстановимые_ошибки#^int-div-zero]]

Что выведет этот код?
```go
func inf() { inf() }
func main() {
    defer func() { recover() }()
    inf()
}
```
?
`fatal error: stack overflow` — программа крашится. recover не помогает при stack overflow, это fatal error уровня рантайма.
![[Восстановимые_и_невосстановимые_ошибки#^stack-overflow]]

В чём разница между поведением `1/0` (int) и `1.0/0.0` (float) в Go?
?
![[Восстановимые_и_невосстановимые_ошибки#^int-div-zero]] + ![[Восстановимые_и_невосстановимые_ошибки#^float-div-zero]]
