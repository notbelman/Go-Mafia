---
type: question
companies:
topic: Rust
subtopic: Смарт-поинтеры
title: Расскажи про RwLock<T> (std)?
---
Reader-Writer Lock - много читателей ИЛИ один писатель.

## Зачем

Оптимизация когда читаешь часто, пишешь редко. Читатели не блокируют друг друга.

## Под капотом
```rust
struct RwLock<T> {
    inner: sys::RwLock,   // OS примитив (pthread_rwlock)
    poison: Flag,
    data: UnsafeCell<T>,
}
```

Состояния:
- Свободен - никто не держит
- Read mode - N читателей, писатели ждут
- Write mode - 1 писатель, все ждут

## Как работает
```
read():
  если write mode → ждать
  иначе → инкремент читателей, вернуть guard

write():
  если кто-то держит (read или write) → ждать
  иначе → войти в write mode, вернуть guard
```

## Writer starvation

Читатели приходят постоянно - писатель ждёт вечно:
```
Reader 1: read lock ✓
Reader 2: read lock ✓
Writer:   хочу write, жду...
Reader 3: read lock ✓  (пока 1,2 держат - можно)
Reader 1: отпустил
Reader 4: read lock ✓
Writer:   всё ещё жду...
```

RwLock даёт read lock пока нет активного writer. Если читатели приходят быстрее чем уходят - writer голодает.

## Poisoning

Как у Mutex - паника с локом отравляет:
```rust
let guard = rwlock.read().unwrap();  // Result
let guard = rwlock.write().unwrap(); // Result
```

## Ограничения

- Дороже Mutex если пишешь часто
- Writer starvation
- Нельзя держать через `.await`

## Когда Mutex vs RwLock

| Сценарий | Выбор |
|----------|-------|
| Много записи | Mutex |
| read >> write | RwLock |
| Не уверен | Mutex |

RwLock сложнее внутри - если write частый, Mutex быстрее.

## Примеры
```rust
let data = Arc::new(RwLock::new(vec![1, 2, 3]));

// Много читателей одновременно
let r1 = data.read().unwrap();
let r2 = data.read().unwrap(); // ок

drop(r1);
drop(r2);

// Писатель - эксклюзивно
let mut w = data.write().unwrap();
w.push(4);
```