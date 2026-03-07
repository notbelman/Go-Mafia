---
type: question
companies:
topic: Rust
subtopic: Смарт-поинтеры
title: Расскажи про tokio::synс::Mutex<T>?
---
Асинхронный mutex - можно держать через `.await`.

## Зачем

Мутабельный доступ к данным в async коде. std::Mutex нельзя держать через await - дедлок.

## Почему std::Mutex нельзя через .await
```rust
let guard = std_mutex.lock().unwrap();
some_async_fn().await;  // поток отдан runtime
// другая задача на этом же потоке хочет lock() - дедлок
```

std::Mutex блокирует поток. В async один поток = много задач. Заблокировал поток - заблокировал всех.

## Под капотом
```rust
struct Mutex<T> {
    s: Semaphore,         // async примитив синхронизации
    data: UnsafeCell<T>,
}
```

При `lock().await`:
1. Проверяет семафор
2. Занят? Task засыпает, добавляется в очередь
3. Освободился? Runtime будит task
4. Поток не блокируется - выполняет другие задачи

## Очередь ожидающих
```
Task A: держит lock
Task B: lock().await - в очереди
Task C: lock().await - в очереди

Task A: drop(guard)
  → runtime будит Task B
  → Task B получает lock
```

FIFO - кто первый встал в очередь, тот первый получит.

## Нет poisoning

tokio::Mutex не отслеживает паники:
```rust
// std
let guard = mutex.lock().unwrap(); // Result из-за poisoning

// tokio
let guard = mutex.lock().await; // сразу guard
```

## Ограничения

- Дороже std::Mutex - overhead на async
- Если не держишь через `.await` - используй std::Mutex или parking_lot

## Когда что
```rust
// std::Mutex - быстрые синхронные операции, drop сразу
{
    let mut guard = data.lock().unwrap();
    *guard += 1;
} // drop

// tokio::Mutex - держишь через .await
let mut guard = data.lock().await;
async_db_write(*guard).await;
*guard += 1;
```

## Примеры
```rust
use tokio::sync::Mutex;
use std::sync::Arc;

let data = Arc::new(Mutex::new(0));
let data_clone = Arc::clone(&data);

tokio::spawn(async move {
    let mut guard = data_clone.lock().await;
    *guard += 1;
    
    some_async_fn().await; // ок
    
    *guard += 1;
});
```