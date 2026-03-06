#flashcards/errors/defer_cost

Когда вычисляются аргументы функции, переданной в defer — в момент регистрации или в момент выполнения? ? ![[Стоимость и аргументы defer#^defer-args-eval]] ![[Стоимость и аргументы defer#^defer-args-example]]

Чем `defer f(x)` отличается от `defer func() { f(x) }()` с точки зрения захвата значения? ? ![[Стоимость и аргументы defer#^defer-closure-example]]

Что выведет этот код?

```go
func main() {
    x := 10
    defer fmt.Println(x)
    x = 20
    fmt.Println(x)
}
```

? `20` затем `10` — аргумент defer вычислен при регистрации, запомнилось 10. Изменение x на 20 не влияет на defer. ![[Стоимость и аргументы defer#^defer-args-example]]

Что выведет этот код?

```go
func main() {
    x := 10
    defer func() { fmt.Println(x) }()
    x = 20
    fmt.Println(x)
}
```

? `20` затем `20` — замыкание захватывает переменную x по ссылке, видит финальное значение. ![[Стоимость и аргументы defer#^defer-closure-example]]

Что выведет этот код?

```go
for i := 0; i < 3; i++ {
    defer fmt.Println(i)
}
```

? `2`, `1`, `0` — LIFO + аргумент вычисляется при регистрации, каждый defer запомнил своё значение i. ![[Стоимость и аргументы defer#^defer-loop-trap]]

Что выведет этот код?

```go
for i := 0; i < 3; i++ {
    defer func() { fmt.Println(i) }()
}
```

? `3`, `3`, `3` — все замыкания захватывают одну переменную i по ссылке. К моменту выполнения defer'ов цикл завершился, i == 3. ![[Стоимость и аргументы defer#^defer-loop-trap]]

Как defer-замыкание может изменить возвращаемое значение функции? ? ![[Стоимость и аргументы defer#^defer-named-return]]

Что выведет этот код?

```go
func count() (result int) {
    defer func() { result++ }()
    return 0
}
func main() { fmt.Println(count()) }
```

? `1` — return устанавливает result=0, затем defer-замыкание делает result++. Именованный возврат позволяет defer'у изменить результат. ![[Стоимость и аргументы defer#^defer-named-return]]

Какова была стоимость defer до Go 1.14 и после? Что изменилось? ? ![[Стоимость и аргументы defer#^defer-cost-old]] ![[Стоимость и аргументы defer#^defer-cost-new]]

Что такое open-coded defer и каковы его ограничения? ? ![[Стоимость и аргументы defer#^defer-open-coded]] ![[Стоимость и аргументы defer#^defer-open-coded-limit]]

Что вернёт `recover()` если паники не было? ? ![[Стоимость и аргументы defer#^recover-no-panic-example]]

Заполни таблицу: что видит defer в трёх случаях — `defer f(x)`, `defer func() { f(x) }()`, `defer func() { result++ }()`? ? ![[Стоимость и аргументы defer#^defer-eval-summary]]