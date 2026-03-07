---
type: question
companies:
topic: Rust
subtopic: Смарт-поинтеры
title: Расскажи про tokio::sync::RwLock<T>?
---
Асинхронный reader-writer lock - можно держать через `.await`.

## Зачем

Много читателей / один писатель в async коде.

## Почему не std::RwLock в async
```rust
let guard = std_rwlock.read().unwrap();
some_async_fn().await;  // поток ушёл к другой задаче
// другая задача хочет write() на этом же потоке - дедлок
```

## Под капотом
```rust
struct RwLock<T> {
    s: Semaphore,         // async примитив
    data: UnsafeCell<T>,
}
```

При `read().await` / `write().await`:
1. Проверяет семафор
2. Занят? Task в очередь, поток свободен
3. Освободился? Runtime будит task

## Отличия от std::RwLock

| | std::RwLock | tokio::RwLock |
|--|-------------|---------------|
| Блокировка | поток спит | task спит, поток работает |
| Poisoning | да | нет |
| Через .await | нельзя | можно |
| Overhead | меньше | больше |

## Write-preferring

tokio::RwLock приоритизирует писателей - меньше writer starvation чем у std:
```
Reader 1: держит read
Writer: хочу write, встал в очередь
Reader 2: хочу read - ждёт! (writer уже в очереди)
Reader 1: отпустил
Writer: получил write
```

Новые читатели ждут если писатель в очереди.

## Ограничения

- Дороже std::RwLock
- Если не держишь через `.await` - используй std или parking_lot

## Когда что
```rust
// std::RwLock - быстрые синхронные операции
{
    let r = data.read().unwrap();
    let val = *r;
} // drop сразу

// tokio::RwLock - держишь через .await
let r = data.read().await;
let result = async_compute(*r).await;
```

## Примеры
```rust
let data = Arc::new(RwLock::new(vec![1, 2, 3]));

// Читатели
let data_clone = Arc::clone(&data);
tokio::spawn(async move {
    let guard = data_clone.read().await;
    async_process(&guard).await;
});

// Писатель
let data_clone = Arc::clone(&data);
tokio::spawn(async move {
    let mut guard = data_clone.write().await;
    guard.push(4);
    async_save(&guard).await;
});
```