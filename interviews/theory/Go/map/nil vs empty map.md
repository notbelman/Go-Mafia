```go
var nilMap map[string]int        // nil — указатель никуда
emptyMap := map[string]int{}     // пустая, hmap создан
alsoEmpty := make(map[string]int)

nilMap == nil    // true
emptyMap == nil  // false
```

^799dc9

 ^nil-vs-empty-decl
[[map Flashcards - nil_vs_empty]]

**Правило:** nil map можно читать, нельзя писать. ^nil-read-only

| Операция           | nil map              | empty map     |
| :----------------- | :------------------- | :------------ |
| `m["key"]`         | ✅ вернёт zero value | ✅ zero value |
| `len(m)`           | ✅ 0                 | ✅ 0          |
| `delete(m, "key")` | ✅ ничего не будет   | ✅ ок         |
| `for range m`      | ✅ 0 итераций        | ✅ 0 итераций |
| `m["key"] = 1`     | ❌ **panic**         | ✅ ок         |
^nil-ops-table

**JSON — важное отличие:** ^nil-json

nil map сериализуется в `"null"`, empty map — в `"{}"`. ^nil-json-null

Если API ожидает `{}` вместо `null` — инициализируй явно или используй `make`. ^nil-json-advice

## Связь
- [[структура hmap]] — nil map = nil указатель на hmap
- [[Map = указатель]] — семантика указателя
- [[операции чтения и вставки]] — шаг 1 вставки: nil → panic
