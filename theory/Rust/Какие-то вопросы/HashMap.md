---
type: question
companies:
topic: Rust
subtopic: Коллекции
title: HashMap - как работает
---
## Структура
- Ключ-значение: HashMap<K, V>
- K должен реализовать Hash + Eq
- По умолчанию SipHash (защита от HashDoS)
## Под капотом
- Массив бакетов
- Хеш ключа → индекс бакета
- Коллизии через open addressing (SwissTable в std)
## Основные операции
```rust
let mut map = HashMap::new();
map.insert("key", 42);        // O(1) амортизированно
map.get("key");               // O(1)
map.remove("key");            // O(1)
map.contains_key("key");      // O(1)
```
## Entry API
```rust
map.entry("key")
   .or_insert(0);             // вставить если нет
   .and_modify(|v| *v += 1);  // изменить если есть
```
## Когда не HashMap
- Нужен порядок → BTreeMap
- Только ключи → HashSet