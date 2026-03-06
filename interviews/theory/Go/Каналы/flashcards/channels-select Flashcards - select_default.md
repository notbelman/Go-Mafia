#flashcards/channels-select/select_default

Что делает `default` в блоке `select`?
?
![[select - default#^select-default-nonblocking]]

Горутина вошла в `select`, ни один канал не готов, есть `default`. Что вызывает рантайм — `gopark` или нет? Почему?
?
![[select - default#^select-default-no-gopark]]

Какую оптимизацию применяет рантайм при захвате мьютексов каналов, если в `select` есть `default`?
?
![[select - default#^select-default-atomic]]

Почему `select` с `default` может использовать атомарные проверки вместо захвата мьютекса?
?
![[select - default#^select-default-atomic]]

Что выведет этот код?
```go
ch := make(chan int)
select {
case v := <-ch:
    fmt.Println("recv", v)
default:
    fmt.Println("no data")
}
```
?
`no data` — канал небуферизован и никто не шлёт, поэтому `case` не готов. `select` с `default` не вызывает `gopark`, немедленно выполняет `default`.
![[select - default#^select-default-no-gopark]]
