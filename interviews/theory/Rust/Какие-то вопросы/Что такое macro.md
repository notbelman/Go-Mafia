---
type: question
companies:
topic: Rust
subtopic: Metaprogramming
title: Что такое macro
---
## Макросы
- Генерация кода на этапе компиляции
- Работают с AST (синтаксическим деревом)
- Вызов с ! : println!, vec!, format!
## Declarative macro (macro_rules!)
```rust
macro_rules! vec {
    ( $( $x:expr ),* ) => {
        {
            let mut temp = Vec::new();
            $( temp.push($x); )*
            temp
        }
    };
}
```
## Procedural macro
- derive макросы: #[derive(Debug, Clone)]
- attribute макросы: #[tokio::main]
- function-like макросы
## Зачем
- Убрать boilerplate
- DRY - не повторяться
- Сериализация, ORM, async runtime