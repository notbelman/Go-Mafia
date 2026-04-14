- `copy(dst, src)` копирует **`min(len(dst), len(src))`** элементов — если dst пустой, ничего не скопируется
- **shallow copy**: примитивы → независимые, указатели → общие данные
- `b := a` **НЕ копирует** — оба смотрят на один array
- независимая копия: `make + copy`, `append([]int(nil), src...)`, `slices.Clone(src)`
- `copy` безопасен при перекрытии: `copy(s[1:], s[:4])` — ок

---
[[slice Flashcards - copy]]
## Функция copy()

`copy(dst, src)` копирует ровно `min(len(dst), len(src))` элементов и возвращает это количество. ^copy-min-len

Если `dst` имеет нулевую длину — ничего не скопируется, даже если cap > 0. ^copy-empty-dst

```go
n := copy(dst, src)  // возвращает кол-во скопированных элементов
```

```go
src := []int{1, 2, 3, 4, 5}
dst := make([]int, 3)

n := copy(dst, src)
fmt.Println(n)    // 3
fmt.Println(dst)  // [1 2 3]
```

## Shallow copy — только значения

Для примитивов `copy` создаёт полностью независимые копии — изменение dst не влияет на src. ^copy-shallow-primitives

Для указателей `copy` копирует сами указатели, не данные за ними. Оба slice указывают на те же объекты. ^copy-shallow-pointers

```go
// Для примитивов — полная копия
src := []int{1, 2, 3}
dst := make([]int, len(src))
copy(dst, src)
dst[0] = 999
fmt.Println(src)  // [1 2 3] — не изменился

// Для указателей — копируются указатели!
type User struct{ Name string }
src := []*User{{"Alice"}, {"Bob"}}
dst := make([]*User, len(src))
copy(dst, src)

dst[0].Name = "Changed"
fmt.Println(src[0].Name)  // "Changed" — изменился!
```

## Присваивание slice — не копирует данные

`b := a` — копирует только дескриптор (ptr, len, cap). Оба slice указывают на один underlying array. ^copy-assign-no-copy

```go
a := []int{1, 2, 3}
b := a        // b и a указывают на один array!
b[0] = 999
fmt.Println(a)  // [999 2 3]
```

## Как сделать независимую копию

Три способа создать независимую копию слайса:

1. `make([]T, len(src))` + `copy(dst, src)` — явно и понятно
2. `append([]int(nil), src...)` — однострочник
3. `slices.Clone(src)` — идиоматично, Go 1.21+

```go
// Способ 1: make + copy
dst := make([]int, len(src))
copy(dst, src)

// Способ 2: append к nil slice
dst := append([]int(nil), src...)

// Способ 3: slices.Clone (Go 1.21+)
dst := slices.Clone(src)
```

## copy с перекрывающимися slice

`copy` корректно обрабатывает перекрывающиеся (overlapping) срезы — безопасен для `copy(s[1:], s[:4])`. ^copy-overlap-safe

```go
s := []int{1, 2, 3, 4, 5}
copy(s[1:], s[:4])  // безопасно! copy корректно обрабатывает overlap
fmt.Println(s)      // [1 1 2 3 4]
```

## Связь
- [[Slice передача в функцию]] — почему присваивание не копирует
- [[Slice утечки памяти]] — копирование для избежания утечек
- [[Slice от slice баги]] — проблемы при шаринге array
