#flashcards/functions/calling_conventions

Что такое calling convention? Какие два главных вопроса она определяет?
?
![[Calling conventions и Go ABI#^cc-definition]]

Чем stack-based calling convention отличается от register-based на уровне механики?
?
![[Calling conventions и Go ABI#^cc-diff-mechanism]]

Какой calling convention использовал Go до 1.17 и что изменилось в 1.17? Конкретные цифры.
?
![[Calling conventions и Go ABI#^cc-go-history]]

На сколько процентов register-based ABI быстрее stack-based в Go 1.17 на amd64? Это синтетика или реальный код?
?
![[Calling conventions и Go ABI#^cc-benchmark]]

Почему даже в register-based ABI часть аргументов всё равно идёт через стек?
?
![[Calling conventions и Go ABI#^cc-register-limit]]

Что произойдёт если функция имеет 10+ аргументов в Go 1.17+?
?
![[Calling conventions и Go ABI#^cc-register-spill]]

В чём разница между stdcall, cdecl и fastcall — кто чистит стек и как передаются аргументы?
?
![[Calling conventions и Go ABI#^cc-who-cleans]]

Совпадает ли Go calling convention с C calling convention?
?
![[Calling conventions и Go ABI#^cc-who-cleans]]

Что выведет этот код?
```go
func main() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("recovered:", r)
        } else {
            fmt.Println("no panic")
        }
    }()
    var buf [1_000_000_000]byte
    _ = buf
    infiniteRecurse()
}
func infiniteRecurse() { infiniteRecurse() }
```
?
Программа упадёт с `fatal error: stack overflow` — recover не вызовется совсем. Stack overflow это не panic, это fatal error, которую нельзя перехватить через recover. Процесс завершится аварийно.
![[Calling conventions и Go ABI#^cc-stackoverflow-fatal]]

Какой максимальный размер стека горутины в Go? Различается ли для 32-бит и 64-бит?
?
![[Calling conventions и Go ABI#^cc-stackoverflow]]

Почему stack overflow нельзя перехватить через recover?
?
![[Calling conventions и Go ABI#^cc-stackoverflow-fatal]]
