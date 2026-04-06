
```go
type slice struct {
    array unsafe.Pointer  // указатель на underlying array
    len   int             // длина (доступные элементы)
    cap   int             // capacity (размер underlying array от ptr)
}
```

^slice-struct-fields

```

slice header (24 байта на 64-bit)
┌─────────────┐
│ array ──────┼──────► [ 1 | 2 | 3 | 4 | 5 ]  underlying array
│ len = 3     │         ↑
│ cap = 5     │         ptr указывает на первый элемент
└─────────────┘
```

^slice-header-diagram

[[slice Flashcards - structure]]

Размер header — **24 байта** на 64-bit платформе (3 поля × 8 байт). ^slice-header-size

`len` — количество элементов в слайсе, доступных по индексу. `cap` — количество элементов от указателя `ptr` до конца underlying array. ^slice-len-cap-meaning

## Связь
- [[len vs cap]] — что значат len и cap
- [[Slice vs Array]] — отличие от массива
- [[Slice передача в функцию]] — копируется header, не array
