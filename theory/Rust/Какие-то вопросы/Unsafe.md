---
type: question
companies:
topic: Rust
subtopic: Safety
title: unsafe - зачем нужен
---
## Что разрешает unsafe
- Разыменование сырых указателей (*const T, *mut T)
- Вызов unsafe функций
- Доступ к mutable static
- Реализация unsafe trait (Send, Sync)
- Работа с FFI (C код)
## Пример
```rust
let ptr = &x as *const i32;
unsafe {
    println!("{}", *ptr);
}
```
## Что НЕ отключает
- Borrow checker работает
- Lifetime проверки работают
- Просто расширяет возможности
## Зачем нужен
- FFI с C/C++
- Низкоуровневые оптимизации
- Реализация safe абстракций (Vec, String внутри unsafe)
## Правило
unsafe код должен гарантировать safety invariants для внешнего safe кода