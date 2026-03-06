- `var s []int` → **nil slice**: == nil true, JSON → `null` ^nil-vs-empty-nil-def
- `s := []int{}` → **empty slice**: == nil false, JSON → `[]` ^nil-vs-empty-empty-def
- **len и cap у обоих = 0**, append работает для обоих ^nil-vs-empty-len-cap
- проверяй `len(s) == 0` вместо `s == nil` — ловит оба случая ^nil-vs-empty-check
- `reflect.DeepEqual(nil, []int{})` → **false** — ломает тесты ^nil-vs-empty-deepequal

---
[[slice Flashcards - nil_vs_empty]]
## Создание

```go
var nilSlice []int           // nil slice
emptySlice := []int{}        // empty slice
emptySlice2 := make([]int, 0) // тоже empty slice
```

## Сравнение

|                  | nil slice | empty slice   |
| :--------------- | :-------- | :------------ |
| == nil           | `true`    | `false`       |
| `len()`          | 0         | 0             |
| `cap()`          | 0         | 0             |
| underlying array | нет       | есть (пустой) |
| JSON             | `null`    | `[]`          |

Ключевое различие: nil slice не имеет underlying array вообще, empty slice — имеет пустой массив. ^nil-vs-empty-underlying

```go
var nilSlice []int
emptySlice := []int{}

fmt.Println(nilSlice == nil)   // true
fmt.Println(emptySlice == nil) // false

fmt.Printf("%#v\n", nilSlice)   // []int(nil)
fmt.Printf("%#v\n", emptySlice) // []int{}
```

## Когда важно различие: JSON

```go
type Response struct {
    Items []string `json:"items"`
}

// nil slice → null
r1 := Response{Items: nil}
json.Marshal(r1)  // {"items":null}

// empty slice → []
r2 := Response{Items: []string{}}
json.Marshal(r2)  // {"items":[]}
```

nil slice сериализуется в JSON как `null`, empty slice — как `[]`. Это критично для API-контрактов. ^nil-vs-empty-json

## Когда важно: reflect.DeepEqual

```go
var nilSlice []int
emptySlice := []int{}

reflect.DeepEqual(nilSlice, emptySlice)  // false!

// В тестах может сломаться assertion
```

`reflect.DeepEqual` различает nil и empty slice — это частая причина неожиданных падений тестов. ^nil-vs-empty-deepequal-detail

## Рекомендация Go Wiki

> Prefer nil slices over empty slices.

```go
// ✅ Идиоматично
var s []int
return nil  // для возврата пустого slice

// ❌ Избыточно
s := []int{}
s := make([]int, 0)
```

Go Wiki рекомендует предпочитать nil slice — меньше аллокаций, нет лишних различий. ^nil-vs-empty-wiki

## Исключение: когда нужен empty slice

```go
// JSON API требует [] вместо null
items := []string{}

// Инициализация карты со slice
m := map[string][]int{
    "a": {},  // нужен non-nil slice
}
```

Исключение из рекомендации: когда JSON-контракт требует `[]`, или когда нужен non-nil slice явно. ^nil-vs-empty-exception

## Связь
- [[Slice структура]] — zero value среза = nil
- [[Slice — не comparable]] — сравнение только с nil
