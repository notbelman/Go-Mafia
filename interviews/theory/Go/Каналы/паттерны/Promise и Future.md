- Promise (JS-стиль) — async task + then(onSuccess, onError): колбэки по результату
- Future (C++-стиль) — блокирующий Get() ждёт результат async задачи
- Promise + Future — обещание (Set) и расписка (Get): promise выполняет, future ждёт
- Всё реализуется поверх каналов, но абстрагирует от них пользователя

---

## Promise (JS-стиль)

"Выполни задачу, потом вызови колбэк": ^promise-js-def

```go
type Promise struct {
    done  chan struct{}
    value interface{}
    err   error
}

func NewPromise(task func() (interface{}, error)) *Promise {
    p := &Promise{done: make(chan struct{})}
    go func() {
        defer close(p.done)
        p.value, p.err = task()
    }()
    return p
}

func (p *Promise) Then(success func(interface{}), failure func(error)) {
    <-p.done  // блокируемся до завершения task
    if p.err == nil {
        success(p.value)
    } else {
        failure(p.err)
    }
}
```

Then можно сделать неблокирующим — обернуть в горутину. ^promise-then-nonblock

## Future (C++-стиль)

"Дай мне значение, когда будет готово": ^future-def

```go
type Future struct {
    result chan interface{}
}

func NewFuture(task func() interface{}) *Future {
    f := &Future{result: make(chan interface{}, 1)}
    go func() {
        f.result <- task()
        close(f.result)
    }()
    return f
}

func (f *Future) Get() interface{} {
    return <-f.result  // блокируемся до результата
}
```

## Promise + Future (C++-стиль)

Разделение ролей: кто **выполняет** обещание и кто **ждёт** результат. ^promise-future-roles

```go
type Promise struct {
    ch      chan interface{}
    settled bool
}

func (p *Promise) Set(val interface{}) {
    if p.settled { return }
    p.settled = true
    p.ch <- val
    close(p.ch)
}

func (p *Promise) GetFuture() *Future {
    return &Future{result: p.ch}
}
```

Аналогия: я **обещаю** принести торт (Promise.Set). Даю тебе **расписку** (Future). Ты **ждёшь** по расписке (Future.Get). ^promise-future-analogy

## Зачем, если есть каналы

- Абстракция: пользователь не знает про каналы
- Типизация: Future[T] явнее чем chan interface{}
- Семантика: код читается как "задача в будущем", а не "канал" ^promise-why

## Связь
- [[Generator]] — тоже абстракция поверх каналов
- [[Error Group]] — тоже async task, но для группы
- [[Done channel]] — future.Get ≈ <-closedCh
