---
type: question
companies:
topic: Rust
subtopic: tokio
title: Что происходит во время рантайма под капотом у tokio?
---
## Основа - event loop

Бесконечный цикл который:
1. Проверяет готовность I/O (epoll/kqueue/iocp)
2. Будит готовые задачи
3. Выполняет их до следующего `.await`

## Future и poll

Каждая async fn компилируется в стейт-машину с методом `poll()`:
- `Poll::Ready(val)` - готово, вот результат
- `Poll::Pending` - не готово, разбуди когда будет

## Waker

Механизм пробуждения. Когда I/O готов - вызывается waker - задача ставится в очередь на выполнение.

## Упрощённо
```
loop {
    // 1. Ждём события от ОС
    let events = epoll.wait();
    
    // 2. Будим задачи связанные с событиями
    for event in events {
        event.waker.wake();
    }
    
    // 3. Выполняем готовые задачи
    while let Some(task) = queue.pop() {
        task.poll(); // выполняется до .await
    }
}
```

## Многопоточность

tokio по дефолту - work-stealing thread pool. Несколько потоков, задачи распределяются между ними.