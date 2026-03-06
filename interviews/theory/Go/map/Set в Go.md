[[map Flashcards - set]]
В Go нет встроенного set. Имитируется через `map[T]struct{}`: ^set-pattern

```go
set := map[string]struct{}{}

set["apple"] = struct{}{}          // добавить

if _, ok := set["apple"]; ok {     // проверить
}

delete(set, "apple")               // удалить
```
^set-code

Почему `struct{}` а не `bool`: `struct{}` занимает 0 байт, `bool` — 1 байт на каждый ключ. ^set-struct-zero

## Связь
- [[Ключи map]] — какие типы можно использовать в set
- [[Память и утечки map]] — struct{} экономит память в бакетах
