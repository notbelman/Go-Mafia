---
type: question
companies:
topic: Rust
subtopic: trait
title: Как происходят вызовы методов у trait object?
---
## Что такое trait object

Fat pointer из двух частей:
- Указатель на данные
- Указатель на vtable

## Что такое vtable

Virtual table - массив указателей на функции. Генерится в compile time для каждой пары `impl Trait for Type`.
```rust
impl Drawable for Circle  → своя vtable
impl Drawable for Square  → своя vtable
```

## Что лежит в vtable
```
[
    drop,    // деструктор
    size,    // размер типа
    align,   // выравнивание
    draw,    // адрес метода draw
    area,    // адрес метода area
]
```

## Как происходит вызов
```rust
let shape: &dyn Drawable = &circle;
shape.draw();

// 1. Взять vtable из fat pointer
// 2. Найти draw в таблице (по индексу)
// 3. Вызвать функцию по адресу
```

## Цена

- Косвенный вызов через указатель
- Нет инлайнинга
- Но один код для всех типов