#flashcards/structs/closer

Какую проблему решает паттерн Closer и почему простой defer не справляется?
?
![[Closer паттерн#^closer-usage]]

Опиши структуру примитивной реализации Closer: поля, методы, сигнатуры.
?
![[Closer паттерн#^closer-impl]]

В каком порядке Closer вызывает зарегистрированные cleanup-функции?
?
![[Closer паттерн#^closer-order]]

Что произойдёт если использовать finalizer вместо Closer для закрытия DB и сетевых соединений при завершении программы?
?
![[Closer паттерн#^closer-guarantee]]

Чем Closer принципиально отличается от `runtime.SetFinalizer` / `runtime.AddCleanup`? Назови все отличия.
?
![[Closer паттерн#^closer-vs-finalizer]]

Что выведет этот код?
```go
type Resource struct{ name string }

func (r *Resource) Close() error {
    fmt.Println("closing", r.name)
    return nil
}

closer := &Closer{}
closer.Add((&Resource{"db"}).Close)
closer.Add((&Resource{"server"}).Close)
closer.Add((&Resource{"worker"}).Close)
closer.Close()
```
?
```
closing db
closing server
closing worker
```
Closer вызывает функции в порядке регистрации (FIFO). Первой зарегистрировали db — она закроется первой.
![[Closer паттерн#^closer-order]]

Почему в Closer нет смысла использовать `defer` внутри метода `Close` для каждой cleanup-функции?
?
![[Closer паттерн#^closer-impl]]
