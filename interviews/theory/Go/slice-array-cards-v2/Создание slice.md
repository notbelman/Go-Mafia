```go
// Все создают slice header + underlying array:
s1 := make([]int, 3, 5)     // len=3, cap=5
s2 := []int{1, 2, 3}        // len=3, cap=3
s3 := array[1:4]            // len=3, cap зависит от array
```
^create-ways

**Три способа создания slice:** `make([]T, len, cap)` — явное задание len и cap; литерал `[]T{...}` — len и cap равны числу элементов; slice от array/slice `arr[i:j]` — len=j-i, cap зависит от исходного. ^create-three

[[slice Flashcards - create]]

```go
// slice от array — ptr указывает внутрь array
arr := [5]int{1, 2, 3, 4, 5}
s := arr[1:3]  // ptr → &arr[1], len=2, cap=4
```
^slice-from-array

**cap при slice от array:** `cap = len(arr) - i`, где `i` — начальный индекс нарезки. Для `arr[1:3]` из массива длиной 5: cap = 5 - 1 = 4. ^cap-formula

**Связь slice и исходного array:** ptr слайса указывает внутрь underlying array. Изменения через slice видны в исходном array и наоборот, пока не произошла реаллокация. ^slice-shares-array

`make([]T, len, cap)` выделяет underlying array размером cap, заполненный zero values для типа T (0 для int, "" для string, false для bool, nil для указателей и т.д.). ^make-zero-values
## Связь
- [[Slice структура]] — что внутри slice header
- [[len vs cap]] — разница между len и cap
- [[Full slice expression]] — создание с ограничением cap
