- map переменная — это **указатель на hmap** (не объект структуры как slice/string) ^map-ptr-def
- передача в функцию: **изменения видны** снаружи (копия указателя → тот же hmap) ^map-ptr-pass
- переприсваивание `m = newMap` внутри функции **не видно** снаружи (локальный указатель) ^map-ptr-reassign
- zero value: **nil** (указатель в никуда). Чтение ок, запись — panic ^map-ptr-nil

---
[[map Flashcards - pointer]]

## Изменение через функцию — видно

```go
func addKey(m map[string]int) {
    m["new"] = 42  // пишем в тот же hmap
}

func main() {
    m := map[string]int{"a": 1}
    addKey(m)
    fmt.Println(m)  // map[a:1 new:42] ← изменилось!
}
```

^map-ptr-visible-code

Копируется указатель, но он указывает на тот же hmap → изменения видны. ^map-ptr-visible-why

## Переприсваивание — не видно

```go
func replaceMap(m map[string]int) {
    m = map[string]int{"x": 100}  // локальный указатель теперь на другой hmap
}

func main() {
    m := map[string]int{"a": 1}
    replaceMap(m)
    fmt.Println(m)  // map[a:1] ← не изменилось!
}
```

^map-ptr-reassign-code

Внутри функции копия указателя переназначена на новый hmap. Оригинальный указатель в main не тронут. ^map-ptr-reassign-why

## Отличие от slice и string

| Тип    | Что копируется         | Размер копии |
| :----- | :--------------------- | :----------- |
| string | объект `{ptr, len}`    | 16 байт      |
| slice  | объект `{ptr, len, cap}` | 24 байта  |
| map    | **указатель** на hmap  | 8 байт       |

^map-ptr-sizes

Slice: append в функции без возврата — потеря данных (копия header). Map: запись в функции — видна снаружи (тот же hmap через указатель). ^map-ptr-vs-slice

## Связь
- [[структура hmap]] — что стоит за указателем
- [[nil vs empty map]] — nil = nil указатель
- [[нельзя взять адрес value]] — семантика указателя и эвакуация
