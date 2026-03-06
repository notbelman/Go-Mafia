---
type: question
companies:
topic: Rust
subtopic: Строки
title: В чем отличие str и String?
---
## str

- Примитивный тип, строковый слайс
- Всегда за ссылкой: `&str`
- Размер неизвестен в compile time (DST)
- Иммутабельный, нельзя изменить

## String

- Owned тип, владеет данными в куче
- Под капотом `Vec<u8>` с гарантией UTF-8
- Можно изменять: push_str, pop, insert...
- Известный размер: указатель + length + capacity

## Конвертация
```rust
let s: &str = "hello";           // литерал - &'static str
let owned: String = s.to_string(); // аллокация

let s: String = String::from("hello");
let slice: &str = &s;            // бесплатно через Deref
```

## Когда что

- `&str` в аргументах - принимает и String, и литералы
- `String` когда владеешь или изменяешь