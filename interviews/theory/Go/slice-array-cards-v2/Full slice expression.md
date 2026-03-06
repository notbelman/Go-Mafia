- `s[i:j:k]` — третий аргумент **ограничивает cap** → `cap = k - i`
- без третьего аргумента cap наследуется до конца underlying array → append может **перезаписать оригинал**
- high и max **нельзя опустить** — Go не хочет угадывать
- главный юзкейс: `a[:2:2]` → append точно создаст новый array

---
[[slice Flashcards - full_slice_expression]]
## Синтаксис

```go
s[i:j:k]
// s[low:high:max]
// len = j - i
// cap = k - i
```

Третий аргумент `k` ограничивает cap результирующего среза: `cap = k - i`. ^fse-syntax

## Что можно опускать

```go
a := []int{1, 2, 3, 4, 5} // len=5, cap=5

// Обычный slice s[i:j] — можно опускать оба
a[:]     // low=0, high=len → len=5, cap=5
a[2:]    // low=2, high=len → len=3, cap=3
a[:3]    // low=0, high=3   → len=3, cap=5

// Full slice s[i:j:k] — можно опускать только low
a[:4:6]  // ✅ low=0         → len=4, cap=6
a[1::6]  // ❌ high нельзя опустить — compile error
a[1:4:]  // ❌ max нельзя опустить  — compile error
a[1:4:5] // ✅ всё указано   → len=3, cap=4
```

В full slice expression `high` и `max` обязательны — Go не хочет угадывать за тебя. Можно опустить только `low`. ^fse-omit-rules

## Зачем нужен full slice expression

Без третьего аргумента `cap` среза наследуется до конца underlying array. Это значит что `append` может писать в элементы оригинала, которые находятся за `len` среза. ^fse-why-needed

## Пример

```go
a := []int{1, 2, 3, 4, 5}

// Обычный slice — cap наследуется
b := a[:2]           // len=2, cap=5
b = append(b, 99)
fmt.Println(a)       // [1 2 99 4 5] ← a[2] ПЕРЕЗАПИСАН

// Full slice — cap ограничен
b := a[:2:2]         // len=2, cap=2
b = append(b, 99)    // cap=2, нужно расширять → новый array
fmt.Println(a)       // [1 2 3 4 5] ← не изменился
```

`a[:2:2]` — cap равен len, поэтому любой `append` создаст новый underlying array и не затронет оригинал. ^fse-pattern-cap-equals-len

## Связь
- [[Slice от slice баги]] — баги при шаринге буфера
- [[Subslicing нарезка]] — базовый синтаксис нарезки
- [[Создание slice]] — способы создания среза
