#flashcards/channels-select/select_algo

Опиши три шага алгоритма `selectgo` в рантайме Go (от входа в `select` до опроса каналов).
?
![[select - алгоритм работы#^select-algo-randomization]]
![[select - алгоритм работы#^select-algo-lock-ordering]]
![[select - алгоритм работы#^select-algo-poll]]

Почему рантайм рандомизирует порядок опроса веток `case` в `select`?
?
![[select - алгоритм работы#^select-algo-why-random]]

Что произойдёт, если убрать рандомизацию из `select`? Какую проблему это вызовет?
?
![[select - алгоритм работы#^select-algo-why-random]]

В каком порядке рантайм захватывает мьютексы каналов в `select`? Почему именно в таком?
?
![[select - алгоритм работы#^select-algo-lock-ordering]]

Почему блокировка каналов в порядке адресов памяти (lock ordering) предотвращает deadlock в `select`?
?
![[select - алгоритм работы#^select-algo-lock-why]]

Что произойдёт на шаге «опрос готовности» в `select`? В каком порядке проверяются каналы?
?
![[select - алгоритм работы#^select-algo-poll]]

Что выведет этот код (запусти мысленно 10 раз)?
```go
ch1 := make(chan string, 1)
ch2 := make(chan string, 1)
ch1 <- "one"
ch2 <- "two"
select {
case v := <-ch1:
    fmt.Println(v)
case v := <-ch2:
    fmt.Println(v)
}
```
?
Выведет либо `one`, либо `two` — недетерминированно. Оба канала готовы одновременно, рантайм выбирает случайно после рандомизации порядка опроса (шаг 1 алгоритма `selectgo`).
![[select - алгоритм работы#^select-algo-randomization]]
