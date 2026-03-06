---
type: question
companies:
topic: Rust
subtopic: Pattern matching
title: Pattern matching - что умеет
---
## match
```rust
match value {
    0 => println!("zero"),
    1 | 2 => println!("one or two"),
    3..=9 => println!("three to nine"),
    n if n < 0 => println!("negative"),
    _ => println!("other"),
}
```
## Деструктуризация
```rust
let (x, y) = (1, 2);
let Point { x, y } = point;
let [first, rest @ ..] = arr;
```
## if let / while let
```rust
if let Some(x) = option {
    // только если Some
}
while let Some(x) = iter.next() {
    // пока Some
}
```
## let else
```rust
let Some(x) = option else {
    return;  // обязан diverge
};
```
## Exhaustiveness
Компилятор проверяет что все варианты покрыты