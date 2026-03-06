---
type: question
companies:
topic: Rust
subtopic: Смарт-поинтеры
title: Расскажи про Cow<T>?
---
Clone on Write - либо ссылка, либо владение. Клонирует только при мутации.

## Зачем

Избежать лишних аллокаций. Читаешь - держишь ссылку. Нужно изменить - клонируешь.

## Под капотом
```rust
enum Cow<'a, B: ToOwned> {
    Borrowed(&'a B),
    Owned(<B as ToOwned>::Owned),
}

// Для str:
Cow::Borrowed(&str)      // просто ссылка
Cow::Owned(String)       // владеем данными
```

## ToOwned trait

Связывает borrowed и owned типы:
```rust
impl ToOwned for str {
    type Owned = String;
    fn to_owned(&self) -> String { self.to_string() }
}

impl ToOwned for [T] {
    type Owned = Vec<T>;
    fn to_owned(&self) -> Vec<T> { self.to_vec() }
}
```

## Методы
```rust
// to_mut() - даёт &mut, клонирует если Borrowed
let mut cow: Cow<str> = Cow::Borrowed("hello");
cow.to_mut().push_str(" world"); // клонирование здесь

// into_owned() - возвращает owned значение
let s: String = cow.into_owned();
```

## Когда полезен

Функции которые иногда модифицируют, иногда нет:
```rust
fn process(input: &str) -> Cow<str> {
    if input.contains(' ') {
        Cow::Owned(input.replace(' ', "_")) // аллокация
    } else {
        Cow::Borrowed(input) // zero-cost
    }
}
```

## Ограничения

- T должен реализовать `ToOwned`
- Overhead на проверку варианта enum (минимальный)
- Усложняет типы в сигнатурах