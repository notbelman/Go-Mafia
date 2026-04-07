#flashcards/atomic/memory_ordering

Почему CPU и компилятор переставляют операции, и почему для одного потока это безопасно?
?
![[Memory Ordering#^mem-reorder-problem]]

Что произойдёт в этом коде при переупорядочивании CPU?
```
// Поток 1:     data = "ready"; flag = true
// Поток 2:     if flag { print(data) }
```
Почему Поток 2 может напечатать пустое значение?
?
![[Memory Ordering#^mem-reorder-bug]]

Как atomic операции решают проблему переупорядочивания? Что даёт Store/Load?
?
![[Memory Ordering#^mem-barrier-solution]]

Какую модель memory ordering использует Go для atomic операций?
?
![[Memory Ordering#^mem-go-seq-consistency]]

Что означает sequential consistency применительно к atomic в Go?
?
![[Memory Ordering#^mem-go-simple]]

Store/Load — барьеры. Сформулируй гарантию: что именно гарантирует пара Store/Load?
?
![[Memory Ordering#^mem-store-load-guarantee]]

Что выведет этот код и почему?
```go
var data string
var ready atomic.Bool

go func() {
    data = "hello"
    ready.Store(true)
}()

for !ready.Load() {}
fmt.Println(data)
```
?
`hello` — `Store(true)` является барьером: всё написанное до него (data = "hello") гарантированно видно после `Load()`. Без atomic была бы data race и возможно пустая строка.
![[Memory Ordering#^mem-store-load-guarantee]]
