---
type: question
companies:
topic: Rust
subtopic: trait
title: Что такое trait?
---
## Определение

Набор методов, которые тип обязуется реализовать. Аналог интерфейсов.

## Синтаксис
```rust
trait Drawable {
    fn draw(&self);
    
    // дефолтная реализация
    fn name(&self) -> &str { "shape" }
}

impl Drawable for Circle {
    fn draw(&self) {
        println!("круг");
    }
}
```

## Зачем

- Абстракция поведения
- Полиморфизм через generics и trait objects
- Стандартные трейты: Clone, Debug, Default, Iterator...

## Trait bounds
```rust
fn print<T: Drawable>(item: T) {
    item.draw();
}

// или where синтаксис
fn print<T>(item: T) where T: Drawable {
    item.draw();
}
```