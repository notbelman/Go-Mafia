---
type: question
companies:
topic: Rust
subtopic: Смарт-поинтеры
title: Расскажи про RefCell<T>?
---
Interior mutability - мутабельность через иммутабельную ссылку. Проверки в runtime.

## Зачем

Обойти правила borrowing когда компилятор не может доказать безопасность, но ты знаешь что всё ок.

## Под капотом
```rust
struct RefCell<T> {
    borrow: Cell<BorrowFlag>,  // счётчик borrow'ов
    value: UnsafeCell<T>,      // данные
}

// BorrowFlag:
// 0 = никто не держит
// >0 = N иммутабельных borrow'ов
// -1 = один мутабельный borrow
```

## borrow() и borrow_mut()
```rust
fn borrow(&self) -> Ref<T> {
    // проверяем что нет mut borrow
    if self.borrow.get() >= 0 {
        self.borrow.set(self.borrow.get() + 1);
        Ref { ... }
    } else {
        panic!("already mutably borrowed");
    }
}

fn borrow_mut(&self) -> RefMut<T> {
    // проверяем что вообще никто не держит
    if self.borrow.get() == 0 {
        self.borrow.set(-1);
        RefMut { ... }
    } else {
        panic!("already borrowed");
    }
}
```

## Ref и RefMut
```rust
struct Ref<'b, T> {
    value: &'b T,
    borrow: &'b Cell<BorrowFlag>,
}

impl<T> Drop for Ref<'_, T> {
    fn drop(&mut self) {
        // декремент счётчика
        self.borrow.set(self.borrow.get() - 1);
    }
}
```

При drop guard'а - счётчик обновляется автоматически.

## Почему не Sync
```rust
// Поток 1: borrow.get() == 0, собирается взять mut
// Поток 2: borrow.get() == 0, тоже берёт mut
// Оба получили &mut T - UB
```

Cell не атомарный - data race на счётчике.

## Ограничения

- Не `Send`, не `Sync` - только однопоточный
- Нарушение правил = паника в runtime
- Overhead на проверки (минимальный)

## Примеры
```rust
let data = RefCell::new(5);

*data.borrow_mut() += 1;
println!("{}", data.borrow()); // 6
```

Паника при нарушении:
```rust
let r1 = data.borrow();
let r2 = data.borrow_mut(); // паника!
```

Безопасная версия:
```rust
if let Ok(mut r) = data.try_borrow_mut() {
    *r += 1;
}
```