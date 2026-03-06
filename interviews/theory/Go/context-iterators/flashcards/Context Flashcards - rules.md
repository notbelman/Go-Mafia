#flashcards/context/rules

Почему ctx нельзя хранить в поле структуры?
?
![[Context ошибки и правила#^ctx-not-state]]

Хранить ctx в struct — антипаттерн. Какие исключения допустимы?
?
![[Context ошибки и правила#^ctx-struct-antipattern]]

Где именно нужно проверять отмену контекста — перед каждой функцией или только в определённых местах?
?
![[Context ошибки и правила#^ctx-check-where]]

Почему defer cancel() обязателен? Что утечёт, если не вызвать?
?
![[Context ошибки и правила#^ctx-defer-cancel-why]]

Что произойдёт, если передать nil вместо контекста? Чем заменить?
?
![[Context ошибки и правила#^ctx-nil-panic]]

Почему cancel нужно вызывать там же где создали контекст?
?
![[Context ошибки и правила#^ctx-cancel-locality]]

Что произойдёт, если разорвать цепочку parent-child в дереве контекстов?
?
![[Context ошибки и правила#^ctx-parent-child]]

Для чего предназначен WithValue? Что нельзя в него класть?
?
![[Context ошибки и правила#^ctx-withvalue-scope]]

ctx.Err() вернул ненулевую ошибку. Какие два значения возможны и что означает каждое?
?
![[Context ошибки и правила#^ctx-err-two-values]]

Назови все обязательные правила работы с context и причину каждого.
?
![[Context ошибки и правила#^ctx-rules-table]]

Что выведет этот код и почему?
```go
type Server struct {
    ctx context.Context
}

func NewServer(ctx context.Context) *Server {
    return &Server{ctx: ctx}
}

func (s *Server) Handle() {
    // использует s.ctx
}
```
?
Код компилируется и работает, но это антипаттерн. Если Server живёт дольше одного запроса, все вызовы Handle() будут разделять один ctx — потеряна связь с конкретным вызовом. ctx должен быть аргументом Handle(ctx context.Context).
![[Context ошибки и правила#^ctx-struct-antipattern]]

Что выведет этот код?
```go
ctx, cancel := context.WithTimeout(context.Background(), time.Second)
// defer cancel() забыли
select {
case <-ctx.Done():
    fmt.Println(ctx.Err())
}
```
?
`context deadline exceeded` — timeout сработает через 1 секунду. Но без defer cancel() таймер runtime удерживает ресурсы до истечения timeout даже если мы ушли раньше. Всегда нужен defer cancel().
![[Context ошибки и правила#^ctx-defer-cancel-why]]
