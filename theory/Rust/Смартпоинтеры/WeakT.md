---
type: question
companies:
topic: Rust
subtopic: Смарт-поинтеры
title: Расскажи про Weak<T>?
---
Слабая ссылка - не владеет данными, не увеличивает strong счётчик.

## Зачем

Разорвать циклические ссылки в Rc/Arc. Иначе утечка памяти.

## Проблема циклических ссылок
```rust
struct Node {
    next: Option<Rc<Node>>,
}

let a = Rc::new(Node { next: None });
let b = Rc::new(Node { next: Some(Rc::clone(&a)) });
// Если a.next = Some(Rc::clone(&b)) - цикл
// strong count никогда не станет 0
// память утечёт
```

## Под капотом
```rust
struct RcInner<T> {
    strong: Cell<usize>,  // владельцы
    weak: Cell<usize>,    // слабые ссылки
    data: T,
}

struct Weak<T> {
    ptr: *mut RcInner<T>,
}
```

Два счётчика:
- `strong` - сколько Rc/Arc владеют данными
- `weak` - сколько Weak ссылок

## Когда что освобождается
```
strong = 0 → данные (T) уничтожаются
weak = 0 && strong = 0 → RcInner освобождается
```

Weak держит RcInner живым, но не данные.

## upgrade()
```rust
impl<T> Weak<T> {
    fn upgrade(&self) -> Option<Rc<T>> {
        if strong_count > 0 {
            // инкремент strong
            Some(Rc { ptr: self.ptr })
        } else {
            None  // данные мертвы
        }
    }
}
```

## downgrade()
```rust
let strong = Rc::new(5);
let weak = Rc::downgrade(&strong);  // Weak из Rc
// strong count не изменился
// weak count += 1
```

## Ограничения

- Нет гарантии что данные живы
- Всегда `upgrade()` перед использованием
- Чуть больше памяти (два счётчика)

## Примеры
```rust
let strong = Rc::new(5);
let weak = Rc::downgrade(&strong);

if let Some(val) = weak.upgrade() {
    println!("{}", val); // 5
}

drop(strong);
assert!(weak.upgrade().is_none()); // данные мертвы
```

Разрыв цикла (дерево):
```rust
struct Node {
    parent: RefCell<Weak<Node>>,      // слабая вверх
    children: RefCell<Vec<Rc<Node>>>, // сильная вниз
}
// Дети не держат родителя - цикла нет
```