---
type: question
companies:
topic: Rust
subtopic: Смарт-поинтеры
title: Расскажи про Rc<T>?
---
Reference Counted - несколько владельцев одних данных. Только однопоточный.

## Зачем

Когда нужно несколько владельцев, но непонятно кто умрёт последним.

## Под капотом
```rust
struct Rc<T> {
    ptr: *mut RcInner<T>,
}

struct RcInner<T> {
    strong: Cell<usize>,  // счётчик владельцев
    weak: Cell<usize>,    // счётчик Weak ссылок
    data: T,
}
```

## Clone и Drop
```rust
// clone - инкремент счётчика
impl<T> Clone for Rc<T> {
    fn clone(&self) -> Rc<T> {
        self.inner().strong.set(self.inner().strong.get() + 1);
        Rc { ptr: self.ptr }
    }
}

// drop - декремент, освобождение при 0
impl<T> Drop for Rc<T> {
    fn drop(&mut self) {
        let count = self.inner().strong.get() - 1;
        self.inner().strong.set(count);
        if count == 0 {
            // drop данные и освободить память
        }
    }
}
```

## Почему не Send

Счётчик - обычный `Cell<usize>`, не атомарный:
```rust
self.strong.set(self.strong.get() + 1);
// три операции: read → add → write
// другой поток может влезть между ними
```

Компилятор не даст передать Rc в другой поток.

## Циклические ссылки
```rust
struct Node {
    next: Option<Rc<Node>>,
}

let a = Rc::new(Node { next: None });
let b = Rc::new(Node { next: Some(Rc::clone(&a)) });
// a.next = Some(Rc::clone(&b)); // цикл! память никогда не освободится
```

Решение - `Weak` для обратных ссылок.

## Ограничения

- Только `&T` - данные иммутабельны
- Не `Send` - только однопоточный
- Циклические ссылки = утечка

## Примеры
```rust
let a = Rc::new(5);
let b = Rc::clone(&a); // счётчик: 2
let c = Rc::clone(&a); // счётчик: 3

println!("{}", Rc::strong_count(&a)); // 3
```

Rc + RefCell для мутабельности:
```rust
let data = Rc::new(RefCell::new(5));
*data.borrow_mut() += 1;
```