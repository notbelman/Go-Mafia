- подсрез от большого среза: **10 байт держат 1 ГБ** (один underlying array) ^leak-subslice-tldr
- указатель на элемент: **1 указатель держит весь array** ^leak-ptr-tldr
- `cap` никогда не уменьшается, `slices.Clip` подрезает cap но **НЕ освобождает память** ^leak-clip-no-free
- единственный способ освободить — **перекопировать** в новый slice (`make + copy`) ^leak-fix-copy
- `runtime.GC()` не поможет пока есть хоть одна ссылка на array ^leak-gc-useless

---
[[slice Flashcards - memory_leaks]]
## Утечка #1: подсрез от большого среза

```go
func findSequence(data []byte) []byte {  // data = 1 ГБ
    idx := findStart(data)
    return data[idx : idx+20]  // 20 байт, но держит 1 ГБ!
}
```

Возвращаемый подсрез разделяет underlying array с `data`. Пока возвращённый slice жив — GC не может собрать 1 ГБ. ^leak-subslice-mechanism

Решение — копировать:
```go
result := make([]byte, 20)
copy(result, data[idx:idx+20])
return result  // независимый array, GC заберёт оригинал
```

^leak-subslice-fix

## Утечка #2: указатель на элемент

```go
func findElement(data []int) *int {  // data = 8 ГБ
    for i := range data {
        if data[i] == target {
            return &data[i]  // 1 указатель держит 8 ГБ!
        }
    }
    return nil
}
```

^leak-ptr-mechanism

Возврат указателя на элемент держит жизнь всего underlying array — GC не может собрать 8 ГБ пока жив один `*int`. ^leak-ptr-why

## Утечка #3: вложенные срезы в структурах

```go
type Row struct {
    Data []byte  // каждый Data = 1 МБ
}
big := make([]Row, 1000)
sub := big[:2]  // "всего 2 элемента"
// Но sub держит array из 1000 Row → ~1 ГБ
```

^leak-struct-slice

Подсрез структурного слайса держит весь backing array, включая все `len(big)` элементов — в том числе те, что за пределами `len(sub)`. ^leak-struct-why

## Capacity не уменьшается

```go
s = s[:10]         // len=10, cap=1000000
s = slices.Clip(s) // len=10, cap=10 — но array на миллион жив!
```

^leak-clip-detail

`slices.Clip` меняет только cap в дескрипторе (через three-index slice), но не перемещает данные и не освобождает старый array. ^leak-clip-why

Единственный способ — перекопировать:
```go
s = slices.Clone(s[:10])  // новый array, GC заберёт старый
```

^leak-clone-fix

## Связь
- [[Slice от slice баги]] — баги при шаринге буфера
- [[Slice copy]] — как правильно копировать
- [[Slice аллокация стек и хип]] — где живёт underlying array
- [[GC и slice]] — когда GC может собрать array
