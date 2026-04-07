| amended | Значение                                |
| :------ | :-------------------------------------- |
| `false` | read содержит все ключи, dirty не нужен |
| `true`  | в dirty есть ключи, которых нет в read  |
^amended-table

**Зачем нужен:**

Оптимизация Load(). Если ключа нет в read: ^amended-purpose

- amended=false → сразу return, не берём лок ^amended-false-path
- amended=true → надо проверить dirty под локом ^amended-true-path

```go
func (m *Map) Load(key any) (value any, ok bool) {
    read := m.loadReadOnly()
    e, ok := read.m[key]
    if !ok && read.amended {  // <-- вот тут проверка
        m.mu.Lock()
        e, ok = m.dirty[key]
        m.missLocked()
        m.mu.Unlock()
    }
    // ...
}
```
^amended-load-code
