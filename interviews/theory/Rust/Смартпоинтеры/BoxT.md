---
type: question
companies:
topic: Rust
subtopic: Смарт-поинтеры
title: Расскажи про Box<T>?
---
Указатель на данные в куче. Самый простой смартпоинтер.

## Зачем

- Данные слишком большие для стека
- Рекурсивные типы (иначе бесконечный размер)
- Передать ownership без копирования
- DST (dynamically sized types) - `Box<dyn Trait>`, `Box<[T]>`

## Под капотом
```rust
struct Box<T> {
    ptr: Unique<T>,  // non-null указатель на heap
}
```

- Выделяет память через `Global` аллокатор (malloc)
- `Deref` / `DerefMut` - работает как обычное значение
- `Drop` вызывает деструктор T, потом освобождает память

## Layout в памяти
```
Stack:              Heap:
Box<i32> [ptr] ---> [i32 value]
8 байт              4 байта + padding
```

## Аллокация
```rust
let b = Box::new(5);

// Под капотом:
// 1. alloc::alloc(Layout::new::<i32>()) - выделить память
// 2. ptr::write(ptr, 5) - записать значение
// 3. Box::from_raw(ptr) - обернуть в Box
```

## Проблема с большими данными
```rust
Box::new([0u8; 10_000_000]) // stack overflow!

// Значение создаётся на стеке, потом копируется в heap
// Решение:
vec![0u8; 10_000_000].into_boxed_slice()
```

## Box::leak
```rust
let static_ref: &'static str = Box::leak(Box::new(String::from("hello")));
// Память никогда не освободится
// Полезно для глобальных данных создаваемых в runtime
```

## Когда НЕ нужен Box

- Маленькие Copy типы - дороже чем на стеке
- Когда хватает ссылки
- Vec/String уже сами в heap