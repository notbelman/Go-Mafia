---
type: question
companies:
topic: Rust
subtopic: trait
title: Orphan rule - сиротское правило
---
## Правило
Можно реализовать трейт для типа только если:
- Трейт твой, ИЛИ
- Тип твой
## Нельзя
```rust
// std::fmt::Display - чужой трейт
// Vec<T> - чужой тип
impl Display for Vec<i32> {}  // ошибка компиляции
```
## Можно
```rust
// свой трейт для чужого типа
impl MyTrait for Vec<i32> {}

// чужой трейт для своего типа
impl Display for MyStruct {}
```
## Newtype pattern - обход
```rust
struct Wrapper(Vec<i32>);
impl Display for Wrapper {}  // ок, Wrapper - наш тип
```
## Зачем нужно
- Предотвращает конфликты реализаций
- Два крейта не могут реализовать один трейт для одного типа
- Coherence - одна реализация на комбинацию trait+type