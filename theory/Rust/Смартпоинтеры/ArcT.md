---
type: question
companies:
topic: Rust
subtopic: Смарт-поинтеры
title: Расскажи про Arc<T>?
---
Atomic Reference Counted - как Rc<T>, но потокобезопасный.

## Зачем

Несколько владельцев + доступ из разных потоков.

## Под капотом
```rust
struct Arc<T> {
    ptr: *mut ArcInner<T>,
}

struct ArcInner<T> {
    strong: AtomicUsize,  // атомарный счётчик
    weak: AtomicUsize,
    data: T,
}
```

## Атомарный счётчик - что это значит

При clone/drop используются атомарные операции:
```rust
// clone:
self.strong.fetch_add(1, Ordering::Relaxed);

// drop:
if self.strong.fetch_sub(1, Ordering::Release) == 1 {
    // освобождаем
}
```

`fetch_add` / `fetch_sub` - неделимые операции. Другой поток не может влезть посередине.

## Почему Rc нельзя в многопоточке
```rust
// Rc использует обычный счётчик:
self.strong.set(self.strong.get() + 1);
// три шага: read → add → write
// другой поток может влезть между ними
```

## Memory Ordering

- `Relaxed` - достаточно для инкремента
- `Release` / `Acquire` - при декременте, чтобы другие потоки увидели все записи до drop

## Реализует Send и Sync

- `Send` - можно передать в другой поток
- `Sync` - можно шарить ссылку между потоками
- Rc не реализует ни то, ни другое

## Ограничения

- Только `&T` - данные иммутабельны
- Для мутабельности: `Arc<Mutex<T>>` или `Arc<RwLock<T>>`
- Циклические ссылки = утечка (используй `Weak`)
- Дороже Rc из-за атомиков (синхронизация между ядрами CPU)

## Примеры
```rust
let data = Arc::new(5);

let handles: Vec<_> = (0..3).map(|_| {
    let data = Arc::clone(&data);
    thread::spawn(move || {
        println!("{}", data);
    })
}).collect();

for h in handles {
    h.join().unwrap();
}
```

Arc + Mutex для мутабельности:
```rust
let data = Arc::new(Mutex::new(0));
let data_clone = Arc::clone(&data);

thread::spawn(move || {
    *data_clone.lock().unwrap() += 1;
});
```

