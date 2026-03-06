- слайсы **нельзя** сравнивать ==, только с `nil` ^cmp-no-eq
- нельзя использовать как **ключ в мапе** ^cmp-no-map-key
- для сравнения: `slices.Equal()` (Go 1.21+, быстро) или `reflect.DeepEqual()` (медленно, рекурсивно) ^cmp-methods
- `reflect.DeepEqual` **различает** nil и empty slice ^cmp-deepequal-nil

---
[[slice Flashcards - not_comparable]]

```go
a := []int{1, 2}
b := []int{1, 2}

a == b          // ❌ compile error: slice can only be compared to nil
a == nil        // ✅ ok

map[[]int]bool  // ❌ compile error: invalid map key type

// Для сравнения используй:
slices.Equal(a, b)         // Go 1.21+
reflect.DeepEqual(a, b)    // медленнее, через рефлексию
```

^cmp-examples

### Почему так решили

**Неясная семантика** — сравнивать по указателю (один ли underlying array) или по элементам? Оба варианта интуитивны, поэтому не выбрали ни один неявно. ^cmp-why-semantics

**Мутабельность** — слайс может измениться после вставки в мапу → хеш станет невалидным. Массивы comparable потому что value type — копируются целиком и не мутируют через общий указатель. ^cmp-why-mutability

## Связь
- [[nil slice vs empty slice]] — DeepEqual различает nil и empty
- [[Slice vs Array]] — массивы comparable, слайсы нет
