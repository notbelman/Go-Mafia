---
type: question
companies:
topic: Rust
subtopic: Смарт-поинтеры
title: Расскажи про UnsafeCell<T>?
---
Примитив для interior mutability. Единственный легальный способ мутировать данные через `&T`.

## Зачем нужен

В Rust `&T` означает "данные не изменятся". Компилятор оптимизирует исходя из этого - кэширует, не перечитывает из памяти.

`UnsafeCell` отключает эти оптимизации - говорит компилятору что данные могут измениться.

## Под капотом
```rust
#[repr(transparent)]
struct UnsafeCell<T> {
    value: T,
}

impl<T> UnsafeCell<T> {
    fn get(&self) -> *mut T {
        // &self -> *mut T
        // единственное место в Rust где это легально
        self as *const UnsafeCell<T> as *const T as *mut T
    }
}
```

## Кто использует

Все interior mutability типы построены на нём:
```rust
struct Cell<T>     { value: UnsafeCell<T> }
struct RefCell<T>  { value: UnsafeCell<T>, borrow: Cell<BorrowFlag> }
struct Mutex<T>    { data: UnsafeCell<T>, inner: sys::Mutex }
struct RwLock<T>   { data: UnsafeCell<T>, inner: sys::RwLock }
```

## Почему unsafe

`get()` возвращает сырой указатель `*mut T`. Никаких проверок - твоя ответственность не накосячить. Cell/RefCell/Mutex оборачивают это в безопасный API.