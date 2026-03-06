#flashcards/interfaces/itab_cache

Где хранится кэш itab в Go runtime?
?
![[Кэш itab#^itab-cache-global]]

Что является ключом в кэше itab?
?
![[Кэш itab#^itab-cache-key]]

Опиши шаги при первом присваивании `var w io.Writer = myFile{}`
?
![[Кэш itab#^itab-cache-first]]

Что происходит при повторном присваивании той же пары (интерфейс, тип)?
?
![[Кэш itab#^itab-cache-repeat]]

Какова сложность вычисления itab — и почему именно O(n+m)?
?
![[Кэш itab#^itab-calc-complexity]]

Как вычисляется hash-ключ при lookup в itabTable?
?
![[Кэш itab#^itab-cache-hash]]

Какая структура данных используется для кэша itabTable?
?
![[Кэш itab#^itab-cache-structure]]

Что происходит когда тип НЕ реализует интерфейс — кэшируется ли этот результат?
?
![[Кэш itab#^itab-cache-fail]]

Что выведет этот код и сколько раз вычисляется itab?
```go
type MyWriter struct{}
func (m MyWriter) Write(p []byte) (int, error) { return len(p), nil }

func main() {
    for i := 0; i < 3; i++ {
        var w io.Writer = MyWriter{}
        _ = w
    }
}
```
?
Ничего не выводит. itab для пары (io.Writer, MyWriter) вычисляется **один раз** на первой итерации, кладётся в itabTable. Вторая и третья итерации — O(1) lookup.
![[Кэш itab#^itab-cache-first]]
![[Кэш itab#^itab-cache-repeat]]
