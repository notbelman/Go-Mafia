---
type: question
companies:
topic: Rust
subtopic: Замыкание
title: Расскажи про Fn/FnMut/FnOnce
---
# `Fn`, `FnMut`, `FnOnce` - три трейта и их иерархия

Три трейта определяют как closure может быть вызвана. Компилятор автоматически реализует максимально гибкий набор трейтов на основе того, что closure делает с захваченными данными.

## Зачем

Разделение на три трейта позволяет:
- Указывать в сигнатурах функций минимальные требования к принимаемым closure
- Компилятору оптимизировать вызовы (статическая диспетчеризация)
- Контролировать сколько раз и как closure может быть вызвана

## Под капотом

**Определения трейтов (упрощённо):**
```rust
// Можно вызвать один раз, потребляет self
trait FnOnce<Args> {
    type Output;
    fn call_once(self, args: Args) -> Self::Output;
}

// Можно вызывать многократно, требует &mut self
trait FnMut<Args>: FnOnce<Args> {
    fn call_mut(&mut self, args: Args) -> Self::Output;
}

// Можно вызывать многократно, требует только &self
trait Fn<Args>: FnMut<Args> {
    fn call(&self, args: Args) -> Self::Output;
}
```

**Иерархия наследования:**
```
    Fn
    │
    ▼
  FnMut
    │
    ▼
  FnOnce

Fn ⊂ FnMut ⊂ FnOnce

Любая Fn автоматически является FnMut и FnOnce
Любая FnMut автоматически является FnOnce
```

**Какой трейт реализуется:**

| Что делает closure с захваченными данными | Реализует |
|-------------------------------------------|-----------|
| Только читает (`&self`) | `Fn` + `FnMut` + `FnOnce` |
| Мутирует (`&mut self`) | `FnMut` + `FnOnce` |
| Перемещает/потребляет (`self`) | Только `FnOnce` |

**Алгоритм компилятора:**
```
если closure перемещает захваченные данные при вызове:
    реализовать только FnOnce
иначе если closure мутирует захваченные данные:
    реализовать FnMut (и FnOnce автоматически)
иначе:
    реализовать Fn (и FnMut, FnOnce автоматически)
```

**Примеры каждого типа:**
```rust
// Fn - только читает
let x = 10;
let fn_closure = || x + 1;
// Структура: { x: &i32 }
// call(&self) -> просто читает *self.x

// FnMut - мутирует
let mut count = 0;
let mut fnmut_closure = || {
    count += 1;
    count
};
// Структура: { count: &mut i32 }
// call_mut(&mut self) -> модифицирует *self.count

// FnOnce - потребляет
let s = String::from("hello");
let fnonce_closure = || {
    drop(s);  // s перемещена и уничтожена
};
// Структура: { s: String }
// call_once(self) -> забирает self.s
```

## Ключевые детали

**Почему иерархия именно такая:**

`Fn: FnMut` потому что если можно вызвать с `&self`, то можно и с `&mut self` (просто не используем мутабельность).

`FnMut: FnOnce` потому что если можно вызвать с `&mut self`, то можно и с `self` (владеем - значит можем взять `&mut`).
```rust
fn call_with_fn<F: Fn()>(f: F) { f(); }
fn call_with_fnmut<F: FnMut()>(mut f: F) { f(); }
fn call_with_fnonce<F: FnOnce()>(f: F) { f(); }

let closure = || println!("hi");  // реализует Fn

call_with_fn(closure);      // OK
call_with_fnmut(closure);   // OK (Fn: FnMut)
call_with_fnonce(closure);  // OK (Fn: FnOnce)
```

**Выбор трейта в сигнатуре функции:**
```rust
// Принимаем максимально широко - FnOnce
fn execute_once<F: FnOnce() -> i32>(f: F) -> i32 {
    f()  // вызываем один раз
}

// Нужно вызвать несколько раз с мутацией
fn execute_twice<F: FnMut() -> i32>(mut f: F) -> i32 {
    f() + f()
}

// Нужно вызвать из нескольких мест / потоков
fn execute_parallel<F: Fn() -> i32 + Send + Sync>(f: F) -> i32 {
    // можем передать &f в разные потоки
    f()
}
```

**Правило выбора для API:**
```
Принимаешь closure? Используй самый слабый подходящий трейт:
- FnOnce если вызываешь один раз
- FnMut если вызываешь несколько раз
- Fn если нужен shared access (несколько ссылок, потоки)
```

**`move` НЕ влияет на трейт:**
```rust
let s = String::from("hello");

// move но только Fn (читаем s)
let c1 = move || s.len();  // Fn + FnMut + FnOnce

// move и только FnOnce (потребляем s)
let c2 = move || drop(s);  // только FnOnce

// Трейт определяется действиями в теле, не способом захвата
```

**Почему closure в итераторах обычно `FnMut`:**
```rust
// Сигнатура Iterator::map
fn map<B, F>(self, f: F) -> Map<Self, F>
where
    F: FnMut(Self::Item) -> B

// FnMut потому что:
// 1. Вызывается многократно (для каждого элемента)
// 2. Может нуждаться в мутации состояния (аккумулятор, счётчик)
```
```rust
let mut sum = 0;
let result: Vec<_> = [1, 2, 3].iter()
    .map(|x| {
        sum += x;  // мутируем захваченную переменную
        x * 2
    })
    .collect();
// Работает потому что map принимает FnMut
```

## Ограничения / Когда не использовать

- `FnOnce` нельзя вызвать повторно - попытка вызвать дважды = ошибка компиляции
- `FnMut` требует эксклюзивный доступ - нельзя иметь несколько ссылок одновременно
- `Fn` в многопоточности требует `Send`/`Sync` для захваченных данных

**Типичная ошибка - попытка вызвать FnOnce дважды:**
```rust
fn call_twice<F: FnOnce()>(f: F) {
    f();
    f();  // ОШИБКА: f уже moved при первом вызове
}

// Исправление:
fn call_twice<F: FnMut()>(mut f: F) {
    f();
    f();  // OK
}
```

## Примеры

**Определение какой трейт реализует closure:**
```rust
fn requires_fn<F: Fn()>(f: F) { f(); }
fn requires_fnmut<F: FnMut()>(mut f: F) { f(); }
fn requires_fnonce<F: FnOnce()>(f: F) { f(); }

let x = 5;
let closure_fn = || println!("{}", x);
requires_fn(closure_fn);  // компилируется -> Fn

let mut y = 5;
let closure_fnmut = || y += 1;
// requires_fn(closure_fnmut);  // ОШИБКА
requires_fnmut(closure_fnmut);  // OK -> FnMut

let s = String::from("hello");
let closure_fnonce = || drop(s);
// requires_fn(closure_fnonce);     // ОШИБКА
// requires_fnmut(closure_fnonce);  // ОШИБКА
requires_fnonce(closure_fnonce);    // OK -> FnOnce
```

**Практический пример - callback с состоянием:**
```rust
struct Button {
    on_click: Box<dyn FnMut()>,  // FnMut чтобы callback мог менять состояние
}

impl Button {
    fn click(&mut self) {
        (self.on_click)();
    }
}

fn main() {
    let mut click_count = 0;
    
    let mut button = Button {
        on_click: Box::new(move || {
            click_count += 1;
            println!("Clicked {} times", click_count);
        }),
    };
    
    button.click();  // Clicked 1 times
    button.click();  // Clicked 2 times
}
```