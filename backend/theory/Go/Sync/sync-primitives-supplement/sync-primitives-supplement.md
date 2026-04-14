## Дополнения к sync примитивам

#### [[CAS и CAS loop]]
Compare-And-Swap паттерн, CAS loop, ABA проблема

#### [[Deadlock]]
Дедлок с мьютексами: double lock, lock ordering, go vet

#### [[Livelock и Starvation]]
Livelock — все активны но не продвигаются, starvation — один голодает

#### [[False Sharing]]
Два поля в одной cache line → contention, padding как решение

#### [[WaitGroup копирование и нюансы]]
Нельзя копировать WaitGroup после первого Use, передавать только по указателю

#### [[Практические ошибки с мьютексами]]
Забыть Unlock, defer Unlock в цикле, lock range, copy mutex
