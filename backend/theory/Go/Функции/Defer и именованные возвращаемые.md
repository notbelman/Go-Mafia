- defer **может** модифицировать именованные возвращаемые значения ^dnr-can-modify
- defer **не может** изменить результат через локальную переменную в замыкании ^dnr-cannot-modify
- причина: return сначала копирует результат, потом выполняет defer'ы; именованные — та же память ^dnr-why
- performance: overhead defer минимален; Go 1.14+ — **defer inlining** в простых случаях ^dnr-inlining-intro

---
[[functions Flashcards - defer_named_returns]]
## Именованные — defer может изменить

```go
func calculate(value int) (result int) {
    defer func() { result += value }()  // модифицирует result
    return value + value
}
fmt.Println(calculate(5))  // 15 (не 10!)
// return записал 10 в result → defer добавил 5 → вернулось 15
```

^dnr-named-example

## Локальная переменная — defer НЕ может изменить

```go
func calculate(value int) int {
    result := value + value
    defer func() { result += value }()  // модифицирует локальную копию
    return result
}
fmt.Println(calculate(5))  // 10 (не 15!)
// return СКОПИРОВАЛ result (10) в возвращаемое значение
// defer изменил локальный result — но возвращаемое значение уже зафиксировано
```

^dnr-local-example

## Почему так

Порядок выполнения return:
1. Вычислить выражение
2. Записать в возвращаемое значение
3. Выполнить defer'ы
4. Вернуться в вызывающую функцию

Именованные возвращаемые — **та же переменная**, которую возвращаем. Defer пишет в неё → результат меняется. ^dnr-return-order

Локальная переменная — **копия** уже сделана на шаге 2, defer меняет оригинал, но копия ушла. ^dnr-local-copy

## Defer inlining (Go 1.14+)

```go
// Компилятор может превратить:
defer f.Close()
// ... код ...
return result

// В:
// ... код ...
f.Close()    // ← вызов перед return, без overhead defer
return result
```

Работает только в **простых случаях**: один defer, нет паники, нет динамического количества defer'ов (цикл). Сложные сценарии — обычный defer через runtime. ^dnr-inlining-detail

## Связь
- [[Именованные возвращаемые значения]] — что это и когда использовать
- [[Defer механика и порядок]] — LIFO, привязка к функции
- [[Defer аргументы и ловушки]] — вычисление аргументов
