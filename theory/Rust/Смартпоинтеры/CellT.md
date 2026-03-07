---
type: question
companies:
topic: Rust
subtopic: Смарт-поинтеры
title: Расскажи про Cell<T>?
---
Interior mutability для Copy типов. Без runtime проверок.

## Зачем

Мутабельность через иммутабельную ссылку для простых типов. Легче чем RefCell.

## Под капотом
```rust
struct Cell<T> {
    value: UnsafeCell<T>,  // магия interior mutability
}
```

`UnsafeCell` - единственный легальный способ получить `&mut T` из `&T` в Rust. Все Cell/RefCell/Mutex построены на нём.

## Почему безопасно

Cell не даёт ссылку на данные - только копирует:
```rust
impl<T: Copy> Cell<T> {
    fn get(&self) -> T {
        // копируем значение наружу
        unsafe { *self.value.get() }
    }
    
    fn set(&self, val: T) {
        // перезаписываем целиком
        unsafe { *self.value.get() = val }
    }
}
```

Нет ссылки - нет проблем с borrowing. Никто не держит указатель на старое значение.

## Почему только Copy
```rust
// Если бы Cell работал с String:
let cell = Cell::new(String::from("hello"));
let s1 = cell.get(); // копия? move? хуй знает
let s2 = cell.get(); // use after move?
```

Copy типы копируются побитово, безопасно дублируются.

## Почему не Sync
```rust
// Поток 1: cell.set(10)
// Поток 2: cell.get()
// get может прочитать половину записанного значения
```

Нет атомарности - data race. Для многопоточки есть `AtomicI32`, `AtomicBool` и т.д.

## Zero overhead

- Нет счётчика borrow'ов (в отличие от RefCell)
- Нет проверок в runtime
- Просто чтение/запись памяти

## Ограничения

- Только для `Copy` типов
- Не `Sync` - только однопоточный
- Нельзя получить `&T` или `&mut T` на содержимое

## Примеры
```rust
let data = Cell::new(5);
data.set(10);
println!("{}", data.get()); // 10
```

В структурах:
```rust
struct Counter {
    value: Cell<i32>,
}

impl Counter {
    fn increment(&self) { // &self, не &mut self
        self.value.set(self.value.get() + 1);
    }
}
```

## Cell vs RefCell

| | Cell | RefCell |
|--|------|---------|
| Типы | Copy | любые |
| Даёт ссылку | нет | да |
| Runtime проверки | нет | да |
| Паника | невозможна | при нарушении borrowing |