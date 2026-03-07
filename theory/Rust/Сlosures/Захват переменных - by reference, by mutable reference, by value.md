---
type: question
companies:
topic: Rust
subtopic: Замыкание
title: Какие знаешь захваты переменных?
---
Компилятор Rust автоматически выбирает минимально необходимый способ захвата каждой переменной на основе того, как она используется в теле closure.

## Зачем

Автоматический выбор способа захвата позволяет:
- Не копировать данные без необходимости
- Сохранять возможность использовать переменную после создания closure
- Гарантировать безопасность на этапе компиляции (borrow checker работает и для closure)

## Под капотом

**Три способа захвата (в порядке приоритета компилятора):**

| Способ | Когда выбирается | Поле в структуре | Что происходит с оригиналом |
|--------|------------------|------------------|----------------------------|
| `&T` (immutable borrow) | Только читаем значение | `x: &'a T` | Можно читать, нельзя менять |
| `&mut T` (mutable borrow) | Мутируем значение | `x: &'a mut T` | Нельзя использовать пока closure жива |
| `T` (by value / move) | Перемещаем или closure outlives данные | `x: T` | Владение переходит в closure |

**Алгоритм компилятора:**
```
для каждой переменной V используемой в теле closure:
    если V перемещается (передаётся в функцию by value, возвращается, etc):
        захватить by value (move)
    иначе если V мутируется:
        захватить by mutable reference
    иначе:
        захватить by immutable reference
```

**Пример всех трёх способов:**
```rust
let a = String::from("hello");  // будет захвачена by reference
let mut b = vec![1, 2, 3];      // будет захвачена by mut reference  
let c = String::from("world");  // будет захвачена by value (move)

let mut closure = || {
    println!("{}", a);      // только читаем -> &String
    b.push(4);              // мутируем -> &mut Vec<i32>
    drop(c);                // перемещаем -> String (by value)
};

// Сгенерированная структура:
// struct __Closure<'a> {
//     a: &'a String,
//     b: &'a mut Vec<i32>,
//     c: String,
// }

closure();

println!("{}", a);  // OK - a была захвачена по ссылке
// println!("{:?}", b);  // OK после вызова closure (borrow закончился)
// println!("{}", c);  // ОШИБКА - c перемещена в closure
```

**Частичный захват структур (Rust 2021+):**

До Rust 2021 захватывалась вся структура. Теперь - только используемые поля:
```rust
struct Data {
    name: String,
    value: i32,
}

let data = Data { 
    name: String::from("test"), 
    value: 42 
};

let closure = || {
    println!("{}", data.value);  // используем только value
};

// Rust 2021: захвачено только data.value (&i32)
// Rust 2018: захвачено &Data целиком

println!("{}", data.name);  // OK в Rust 2021, ошибка в Rust 2018
```

## Ключевые детали

**Borrow checker работает для closure так же как для обычного кода:**
```rust
let mut x = 5;

let closure = || {
    x += 1;  // closure захватывает &mut x
};

// x += 1;  // ОШИБКА: x уже mutably borrowed closure'ой

closure();

x += 1;  // OK: closure больше не используется, borrow закончился
```

**Захват по ссылке создаёт лайфтайм зависимость:**
```rust
fn create_closure() -> impl Fn() {
    let x = 5;
    || println!("{}", x)  // ОШИБКА: x не живёт достаточно долго
    // closure захватывает &x, но x умрёт при выходе из функции
}
```

**Важный edge case - захват ссылки на ссылку:**
```rust
let x = 5;
let r = &x;

let closure = || *r;  // захватывает r: &&i32, не x

// closure использует r, не x напрямую
// если бы r был &mut, closure захватила бы &mut &mut i32
```

**Reborrow при захвате `&mut`:**
```rust
let mut x = 5;
let r = &mut x;

let mut closure = || {
    *r += 1;  // closure захватывает &mut ссылку на r, то есть reborrow
};

closure();
// r всё ещё валидна после последнего использования closure
*r += 1;  // OK
```

## Ограничения / Когда не использовать

- Если closure должна пережить scope переменной - захват по ссылке не подойдёт, нужен `move`
- Mutable borrow в closure блокирует переменную на всё время жизни closure (не только на время вызова)
- Нельзя иметь две closure которые обе мутируют одну переменную одновременно

## Примеры

**Демонстрация выбора способа захвата:**
```rust
fn main() {
    let a = 1;                    // Copy type
    let b = String::from("x");    // не Copy
    let mut c = vec![1];          // будем мутировать
    let d = String::from("y");    // будем перемещать
    
    let mut closure = || {
        let _ = a;        // Copy - копируется, но захват всё равно &i32
        let _ = &b;       // явная ссылка, захват &String
        c.push(2);        // мутация, захват &mut Vec
        let _ = d;        // implicit move для не-Copy типа -> захват by value
    };
    
    // a доступна (Copy)
    // b доступна (захвачена по ссылке)
    // c недоступна пока closure жива (mut borrow)
    // d недоступна (moved)
    
    closure();
}
```

**Ошибка из-за конфликта borrows:**
```rust
let mut data = vec![1, 2, 3];

let read = || data.len();           // &Vec
let write = || data.push(4);        // &mut Vec

// read();   // ОШИБКА если write существует
// write();  // &mut и & не могут сосуществовать

// Решение: убедиться что одна closure "умерла" перед созданием другой
```