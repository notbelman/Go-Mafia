---
type: question
companies:
topic: Rust
subtopic: Concurrency
title: Send и Sync
---
## Send
- Тип можно передать в другой поток
- Владение переходит безопасно
- Почти все типы Send
## Sync
- К типу можно обращаться из нескольких потоков через &T
- T: Sync значит &T: Send
## Не Send / не Sync
- Rc<T> - не Send (счётчик не атомарный)
- RefCell<T> - не Sync (runtime borrow checking не потокобезопасен)
- *const T, *mut T - ни то, ни другое
## Arc + Mutex
```rust
let data = Arc::new(Mutex::new(0));
// Arc - потокобезопасный счётчик (Sync + Send)
// Mutex - синхронизация доступа
```
## Автотрейты
Компилятор выводит автоматически, реализовывать вручную - unsafe