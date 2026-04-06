При передаче slice в функцию копируется **header** (24 байта), а **underlying array — НЕТ**. ^slice-pass-copy-header

```
main()                      func(s []int)
┌─────────────┐             ┌─────────────┐
│ array ──────┼─────┬───────┼── array     │  копия header
│ len = 3     │     │       │ len = 3     │
│ cap = 5     │     │       │ cap = 5     │
└─────────────┘     │       └─────────────┘
                    ▼
                  [ 1 | 2 | 3 | _ | _ ]  - один underlying array!
```

^slice-pass-diagram
[[slice Flashcards - pass_to_func]]
## Изменение элементов — видно снаружи

```go
func modify(s []int) {
    s[0] = 999  // меняем underlying array
}

func main() {
    s := []int{1, 2, 3}
    modify(s)
    fmt.Println(s)  // [999 2 3] ✅ изменился
}
```

^slice-pass-mutate-visible

Изменение **элементов** через индекс видно снаружи, потому что обе копии header указывают на один и тот же underlying array. ^slice-pass-why-visible

## append внутри функции — НЕ виден снаружи

Если внутри функции вызвать `append` без реаллокации, он запишет данные в underlying array (видно снаружи через тот же ptr), но **изменит `len` только локальной копии header** — снаружи len не изменится. При реаллокации функция получает новый array, и связь полностью теряется. ^slice-pass-append-invisible

Если нужно вернуть изменённый slice из функции — нужно либо вернуть его явно, либо передавать `*[]T`. ^slice-pass-return-pattern

## Связь
- [[Slice структура]] — что копируется (ptr, len, cap)
- [[append внутри функции — НЕ видно снаружи]] — почему append не виден
- [[Array аллокация и копирование]] — массив копируется ЦЕЛИКОМ
