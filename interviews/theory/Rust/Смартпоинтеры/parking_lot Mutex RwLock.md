---
type: question
companies:
topic: Rust
subtopic: Смарт-поинтеры
title: Расскажи про parking_lot::Mutex::RwLock?
---
Быстрая замена std версий. Drop-in replacement.

## Зачем

Производительность. Быстрее std в большинстве сценариев.

## Под капотом
```rust
// parking_lot::Mutex - всего 1 байт
struct Mutex<T> {
    state: AtomicU8,  // состояние лока
    data: UnsafeCell<T>,
}

// std::Mutex - 40+ байт на Linux
// Оборачивает pthread_mutex_t
```

## Почему быстрее

### Adaptive spinning
```
1. Пробуем lock
2. Занят? Крутимся в цикле несколько раз (spin)
3. Всё ещё занят? Теперь засыпаем
```

std сразу засыпает - дорогой syscall. parking_lot сначала спинит - если lock освободится быстро, syscall не нужен.

### Меньше размер

- parking_lot: 1 байт + size_of::<T>()
- std: 40+ байт + size_of::<T>()

Меньше памяти, лучше cache locality.

## Нет poisoning

std возвращает `Result` - вдруг поток запаниковал с локом:
```rust
// std
let guard = data.lock().unwrap();

// parking_lot - сразу guard
let guard = data.lock();
```

parking_lot считает: паника = баг, чини код, а не обрабатывай.

## Fair unlocking
```rust
use parking_lot::FairMutex;

let mutex = FairMutex::new(0);
```

Обычный Mutex - кто первый проснулся, тот и получил. FairMutex - очередь FIFO, никто не голодает.

## Ограничения

- Внешняя зависимость
- Нет poisoning - если нужен, используй std
- Всё ещё блокирующий - не для async

## Когда что

| Критерий | std | parking_lot | tokio |
|----------|-----|-------------|-------|
| Без зависимостей | ✓ | | |
| Скорость | | ✓ | |
| Poisoning | ✓ | | |
| Async через .await | | | ✓ |

## Примеры
```rust
use parking_lot::{Mutex, RwLock};

let mutex = Mutex::new(0);
*mutex.lock() += 1;

let rwlock = RwLock::new(vec![1, 2, 3]);
println!("{:?}", *rwlock.read());
rwlock.write().push(4);
```