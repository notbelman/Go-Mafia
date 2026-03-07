---
type: question
companies:
topic: Rust
subtopic: Замыкание
title: Расскажи про move
---
`move` перед closure заставляет захватить все используемые переменные по значению (переместить или скопировать), независимо от того, как они используются в теле.

## Зачем

Два основных сценария:

1. **Closure должна пережить scope переменных** - передача в другой поток, возврат из функции, сохранение в структуру с большим лайфтаймом
2. **Явно забрать владение** - даже если по коду достаточно ссылки, хотим переместить данные в closure

## Под капотом

**Без `move`:**
```rust
let s = String::from("hello");
let closure = || println!("{}", s);

// Структура: { s: &String }
// s всё ещё доступна после создания closure
```

**С `move`:**
```rust
let s = String::from("hello");
let closure = move || println!("{}", s);

// Структура: { s: String }
// s перемещена, недоступна после создания closure
```

**Что происходит с разными типами:**

| Тип | Без `move` | С `move` |
|-----|-----------|----------|
| `String`, `Vec<T>`, `Box<T>` | `&T` или `&mut T` | Перемещается (move) |
| `i32`, `f64`, `bool`, `char` | `&T` | Копируется (Copy trait) |
| `&T` (ссылка) | `&&T` | `&T` (копия ссылки) |
| `&mut T` | `&mut &mut T` | `&mut T` (move ссылки) |

**Copy типы с `move`:**
```rust
let x: i32 = 5;
let closure = move || x + 1;

println!("{}", x);  // OK! i32 реализует Copy, x скопирован в closure
```

**Move семантика для не-Copy:**
```rust
let s = String::from("hello");
let closure = move || s.len();

// println!("{}", s);  // ОШИБКА: s перемещена
```

## Ключевые детали

**`move` применяется ко ВСЕМ захваченным переменным:**
```rust
let a = String::from("a");  // хотим переместить
let b = String::from("b");  // хотим оставить

let closure = move || {
    println!("{}", a);
};

// println!("{}", a);  // ОШИБКА: перемещена
// println!("{}", b);  // OK: b не используется в closure, не захвачена
```

Если нужно переместить одну переменную, но оставить другую:
```rust
let a = String::from("a");
let b = String::from("b");

let b_ref = &b;  // создаём ссылку заранее
let closure = move || {
    println!("{}", a);   // a перемещена
    println!("{}", b_ref);  // b_ref (ссылка) скопирована
};

println!("{}", b);  // OK
```

**`move` НЕ влияет на какой Fn-трейт реализует closure:**
```rust
let s = String::from("hello");

// Без move: Fn (только читаем по ссылке)
let c1 = || println!("{}", s);

// С move: всё равно Fn! (только читаем, просто владеем данными)
let c2 = move || println!("{}", s);

// Fn-трейт определяется тем ЧТО делаем с данными, не КАК захватили
```

**Частичный захват и `move` (Rust 2021):**
```rust
struct Data {
    name: String,
    id: i32,
}

let data = Data { name: String::from("test"), id: 1 };

// Без move: захватывает только &data.id
let c1 = || data.id;
println!("{}", data.name);  // OK

// С move: перемещает ВСЮ структуру (Rust 2021 поведение для move)
let c2 = move || data.id;
// println!("{}", data.name);  // ОШИБКА: вся data перемещена
```

Если нужно переместить только часть:
```rust
let data = Data { name: String::from("test"), id: 1 };
let name = data.name;  // вытаскиваем поле заранее

let closure = move || println!("{}", name);
// data.id всё ещё доступен (примитив остался в data)
```

## Ограничения / Когда не использовать

- `move` с большими данными увеличивает размер closure
- Нельзя использовать переменную после `move` closure (если не Copy)
- `move` не спасёт если данные содержат ссылки с коротким лайфтаймом

**`move` не продлевает жизнь ссылок:**
```rust
fn bad() -> impl Fn() {
    let x = 5;
    let r = &x;
    move || println!("{}", r)  // ОШИБКА: r это &i32, указывает на локальную x
    // move копирует ССЫЛКУ, не данные на которые она указывает
}

fn good() -> impl Fn() {
    let x = 5;
    move || println!("{}", x)  // OK: x (i32) скопирован в closure
}
```

## Примеры

**Передача в другой поток (главный use case):**
```rust
use std::thread;

let data = vec![1, 2, 3];

// Без move - ошибка компиляции
// let handle = thread::spawn(|| {
//     println!("{:?}", data);  // data может умереть раньше потока
// });

// С move - OK
let handle = thread::spawn(move || {
    println!("{:?}", data);  // data перемещена в поток
});

handle.join().unwrap();
// data недоступна здесь
```

**Возврат closure из функции:**
```rust
fn make_adder(x: i32) -> impl Fn(i32) -> i32 {
    // x - параметр функции, умрёт при выходе
    // move копирует x (Copy тип) в closure
    move |y| x + y
}

let add5 = make_adder(5);
println!("{}", add5(10));  // 15
```

**Комбинация с Clone для сохранения доступа:**
```rust
let data = vec![1, 2, 3];
let data_clone = data.clone();

let closure = move || {
    println!("{:?}", data_clone);
};

println!("{:?}", data);  // OK: оригинал остался
closure();
```