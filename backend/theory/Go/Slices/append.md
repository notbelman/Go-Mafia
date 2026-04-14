- `len + новые > cap` → **новый array + копирование** (O(n)), иначе O(1) ^append-rule-realloc
- **всегда** `s = append(s, x)` — присваивай результат, иначе потеряешь ^append-assign-required
- рост cap: **< 256 → ×2**, потом плавно к **×1.25** (Go 1.18+) ^append-growth-summary
- амортизированная сложность: **O(1)** — реаллокации редкие, cap растёт экспоненциально ^append-amortized
- `make([]int, 0, n)` — pre-allocate если знаешь размер ^append-prealloc
- append — функция, а не метод: slice безымянный тип, нужен возврат нового header ^append-why-func

---
[[slice Flashcards - append]]

## Как append работает внутри
1. Проверяет: `len + новые > cap`?
2. Если **нет** — записывает элементы в underlying array начиная с позиции `len`, увеличивает `len`
3. Если **да** — выделяет новый массив, копирует старые данные, записывает новые элементы
^append-mechanic
## Когда новый underlying array
```go
s := make([]int, 2, 3)  // len=2, cap=3
s = append(s, 1)        // len=3, cap=3 — влезло в старый array
s = append(s, 2)        // len=4, cap > 3 — НОВЫЙ array!

s = append(s, x)        // ✅ ВСЕГДА присваивай результат
append(s, x)            // ❌ результат потерян
```

**Правило:** `len + новые_элементы > cap` → аллокация нового array + копирование. ^append-rule-detail

Pre-allocate если знаешь размер: `make([]int, 0, 10000)` ^append-prealloc-pattern

## Формула роста capacity (Go 1.18+)
```go
// runtime/slice.go — nextslicecap()
const threshold = 256

if oldCap < threshold {
    newCap = oldCap * 2                    // ×2 для маленьких
} else {
    newCap += (newCap + 3*threshold) >> 2  // плавный переход к ×1.25
}
```

| oldCap | Рост   |
| :----- | :----- |
| < 256  | ×2.0   |
| 512    | ×1.63  |
| 1024   | ×1.44  |
| 4096+  | ≈×1.25 |

Порог перехода — `threshold = 256`. До него cap удваивается, после — растёт по формуле `(newCap + 3*threshold) >> 2`. ^append-growth-formula

## Сложность append

| Случай               | Сложность |
| :------------------- | :-------- |
| Есть место в cap     | O(1)     |
| Нужна реаллокация    | O(n)     |
| **Амортизированная** | **O(1)** |

Амортизированная O(1) — реаллокации редкие и cap растёт экспоненциально. ^append-complexity-why

За N операций append суммарно копируется ≈ 2N элементов → в среднем O(1) на операцию. ^append-complexity-proof

## Почему append — функция, а не метод

1. **Меняет header** — append может вернуть новый `{ptr, len, cap}`, а метод на value receiver не может изменить сам слайс снаружи ^append-func-reason-header
2. **Slice — безымянный тип** — `[]int` не имеет именованного типа, методы можно определять только на именованных типах ^append-func-reason-unnamed
3. **Должна возвращать значение** — дизайн Go: явный `s = append(s, x)` показывает, что слайс мог измениться целиком ^append-func-reason-return

## Связь
- [[Slice структура]] — ptr, len, cap
- [[append внутри функции — НЕ видно снаружи]] — почему append в функции не виден
- [[Slice аллокация стек и хип]] — реаллокация всегда в heap
