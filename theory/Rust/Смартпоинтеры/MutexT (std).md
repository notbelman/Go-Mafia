---
type: question
companies:
topic: Rust
subtopic: Смарт-поинтеры
title: Расскажи про Mutex<T> (std)?
---
Mutual Exclusion - гарантирует что только один поток имеет доступ к данным.

## Зачем

Мутабельный доступ к данным из нескольких потоков.

## Под капотом
```rust
struct Mutex<T> {
    inner: sys::Mutex,      // OS примитив (pthread_mutex / SRWLOCK)
    poison: Flag,           // флаг отравления
    data: UnsafeCell<T>,    // данные
}
```

При `lock()`:
1. Вызов в ОС - поток блокируется если mutex занят
2. ОС будит поток когда mutex свободен
3. Возвращается `MutexGuard`

## MutexGuard
```rust
struct MutexGuard<'a, T> {
    lock: &'a Mutex<T>,
}

impl<T> Deref for MutexGuard<T> { ... }     // *guard читает данные
impl<T> DerefMut for MutexGuard<T> { ... }  // *guard = x пишет данные
impl<T> Drop for MutexGuard<T> {
    fn drop(&mut self) {
        self.lock.inner.unlock();  // автоматический unlock
    }
}
```

## Poisoning

Если поток запаниковал держа lock - данные могут быть в невалидном состоянии:
```rust
let data = Arc::new(Mutex::new(vec![1, 2, 3]));

thread::spawn({
    let data = Arc::clone(&data);
    move || {
        let mut guard = data.lock().unwrap();
        guard.push(4);
        panic!("oops"); // guard не дропнулся нормально
    }
});

// В другом потоке:
let result = data.lock(); // Err(PoisonError)
```

Можно проигнорировать:
```rust
let guard = data.lock().unwrap_or_else(|e| e.into_inner());
```

## Почему нельзя через .await
```rust
let guard = mutex.lock().unwrap();
some_async_fn().await;  // поток отдан другой задаче
*guard += 1;            // другая задача может попробовать lock() - дедлок
```

std::Mutex блокирует поток. В async один поток выполняет много задач - заблокируешь поток, заблокируешь всех.

## Ограничения

- Блокирующий - поток спит пока ждёт
- Нельзя держать через `.await`
- `lock()` возвращает `Result` из-за poisoning

## Примеры
```rust
let data = Arc::new(Mutex::new(0));

let handles: Vec<_> = (0..10).map(|_| {
    let data = Arc::clone(&data);
    thread::spawn(move || {
        let mut guard = data.lock().unwrap();
        *guard += 1;
    })
}).collect();

for h in handles {
    h.join().unwrap();
}
```