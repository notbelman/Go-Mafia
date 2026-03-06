---
type: question
companies:
topic: Rust
subtopic: Типы
title: enum - чем мощнее чем в других языках
---
## Variants с данными
```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    Color(u8, u8, u8),
}
```
## Sum types (алгебраические типы)
- Каждый variant может иметь разные данные
- В C/Go enum - просто числа
- В Rust - полноценные tagged unions
## Методы на enum
```rust
impl Message {
    fn call(&self) {
        match self {
            Message::Write(s) => println!("{}", s),
            _ => {}
        }
    }
}
```
## Стандартные enum
- Option<T> - Some(T) | None
- Result<T, E> - Ok(T) | Err(E)
## Размер в памяти
Размер самого большого варианта + тег (discriminant)