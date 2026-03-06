```go
m := map[string]User{"bob": {Age: 30}}

m["bob"].Age = 31      // ошибка компиляции
ptr := &m["bob"]       // ошибка компиляции

// cannot take the address of m["bob"]
// cannot assign to struct field m["bob"].Age in map
```

^addr-compile-error
[[map Flashcards - addr_value]]

**Почему?**

При evacuation (resize) пары переезжают в новые buckets. Указатель стал бы невалидным: ^addr-why-evacuation

```
до resize:    &m["bob"] → bucket 2, slot 3 → {Age: 30}
после resize: bucket 2, slot 3 → мусор или другой ключ
              {Age: 30} теперь в bucket 5, slot 1
```

^addr-invalidation-example

**Решения:**
```go
// 1. Копия → изменить → записать
u := m["bob"]
u.Age = 31
m["bob"] = u

// 2. Хранить указатели
m := map[string]*User{"bob": {Age: 30}}
m["bob"].Age = 31  // ок, адрес User не меняется, меняется только адрес в map
```

^addr-solutions

## Связь
- [[Evacuation]] — причина: элементы переезжают при resize
- [[операции чтения и вставки]] — алгоритм записи
- [[Map = указатель]] — семантика map как указателя
