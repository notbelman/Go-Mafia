---
type: question
companies:
topic: Rust
subtopic:
  - Смарт-поинтеры
title: Как Box понимает когда освобождать память?
---
## Drop trait

Box выходит из scope → вызывается `drop()` → память освобождена.
```rust
{
    let b = Box::new(5);
} // drop вызван автоматически
```

## RAII

Resource Acquisition Is Initialization.

Владение ресурсом = ответственность за очистку. Компилятор сам вставляет вызовы drop в конце scope.

## Под капотом
```rust
impl<T> Drop for Box<T> {
    fn drop(&mut self) {
        // 1. вызвать drop для T
        // 2. освободить память через деаллокатор
    }
}
```