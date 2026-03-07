---
type: question
companies:
topic: Rust
subtopic: trait
title: Ключевое слово dyn
---
## Что это

`dyn Trait` - trait object, динамическая диспетчеризация.

## Зачем

Когда конкретный тип неизвестен в compile time:
```rust
let shapes: Vec<Box<dyn Drawable>> = vec![
    Box::new(Circle),
    Box::new(Square),
];

for shape in shapes {
    shape.draw(); // какой метод вызвать - решается в runtime
}
```

## Generics vs dyn
```rust
// Generics - статическая диспетчеризация
// Код генерится для каждого типа, быстрее
fn draw<T: Drawable>(item: T)

// dyn - динамическая диспетчеризация
// Один код, выбор метода в runtime, overhead на vtable
fn draw(item: &dyn Drawable)
```

## Когда dyn

- Разные типы в одной коллекции
- Тип определяется в runtime
- Хочешь уменьшить размер бинарника