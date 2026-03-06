---
type: question
companies:
topic: Rust
subtopic: Смарт-поинтеры
title: Отличие Mutex/RwLock
---
## Проблема

Несколько потоков пишут в одни данные = data race:
```rust
// Поток 1: count = count + 1
// Поток 2: count = count + 1
// Результат: хуй знает какой
```

## Mutex

Mutual Exclusion - только один поток имеет доступ:
```rust
let data = Arc::new(Mutex::new(0));

let mut guard = data.lock().unwrap();
*guard += 1;
// drop guard - разблокировка
```

## RwLock

Много читателей ИЛИ один писатель:
```rust
let data = Arc::new(RwLock::new(vec![1, 2, 3]));

// Читатели - не блокируют друг друга
let r1 = data.read().unwrap();
let r2 = data.read().unwrap(); // ок, оба читают

// Писатель - эксклюзивный доступ
let mut w = data.write().unwrap();
w.push(4);
```

## Когда что

- Mutex - просто, надёжно, дефолтный выбор
- RwLock - выгода когда читаешь часто, пишешь редко