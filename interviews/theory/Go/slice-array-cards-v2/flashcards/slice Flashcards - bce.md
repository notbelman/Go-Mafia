#flashcards/slice/bce

Что именно вставляет компилятор Go перед каждым доступом к элементу слайса/массива?
?
![[Bound check elimination#^bce-what-compiler-inserts]]

С какой версии Go появился BCE? Каким флагом проверить где остались bound checks?
?
![[Bound check elimination#^bce-version-flag]]

Почему доступ к элементу в горячем цикле — потенциальная проблема производительности?
?
![[Bound check elimination#^bce-overhead-per-access]]

Почему `s[3]; s[2]; s[1]; s[0]` даёт 1 bound check, а `s[0]; s[1]; s[2]; s[3]` — 4?
?
![[Bound check elimination#^bce-opt-reverse-constants]]

Что выведет этот код? Сколько bound checks в каждом варианте?
```go
s := []int{10, 20, 30, 40}
// Вариант A:
_ = s[3]; _ = s[2]; _ = s[1]; _ = s[0]
// Вариант B:
_ = s[0]; _ = s[1]; _ = s[2]; _ = s[3]
```
?
Оба варианта не паникуют (s достаточно длинный). Разница в количестве bound checks: вариант A — 1, вариант B — 4. Компилятор видит наибольший индекс первым в A и устраняет остальные.
![[Bound check elimination#^bce-opt-reverse-constants]]

Как убрать повторные bound checks при доступе `s[i], s[i+1], s[i+2], s[i+3]` с переменным `i`?
?
![[Bound check elimination#^bce-opt-subslice]]

Почему подсрез `sub := s[i:i+4]` помогает BCE лучше чем прямые `s[i+3]...s[i]`?
?
![[Bound check elimination#^bce-opt-subslice]]

Как убрать bound checks в цикле `for i := 0; i < size; i++ { values[i] *= 2 }`?
?
![[Bound check elimination#^bce-opt-pre-check]]

Что выведет этот код? Есть ли bound check внутри цикла?
```go
func process(values []int, size int) {
    _ = values[size-1]
    for i := 0; i < size; i++ {
        values[i] *= 2
    }
}
process([]int{1, 2, 3, 4, 5}, 5)
```
?
Код выполнится без паники, перемножит все элементы на 2. Фиктивный доступ `values[size-1]` до цикла убирает bound checks внутри — это стандартный BCE-паттерн.
![[Bound check elimination#^bce-opt-pre-check]]

Как `unsafe` позволяет полностью убрать bound checks? Какой главный риск?
?
![[Bound check elimination#^bce-opt-unsafe]]
