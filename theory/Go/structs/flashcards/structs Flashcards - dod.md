#flashcards/structs/dod

Что такое Data Oriented Design? Сформулируй одним предложением суть трансформации данных.
?
![[DOD Data Oriented Design#^dod-layout]]

Почему при OOD итерации по одному полю кэш используется неэффективно? Объясни на уровне кэшлиний.
?
![[DOD Data Oriented Design#^dod-ood-problem]]

Почему при DOD итерации по одному полю кэш используется эффективно?
?
![[DOD Data Oriented Design#^dod-dod-benefit]]

Помимо cache-friendliness, какое ещё преимущество даёт DOD по памяти и почему?
?
![[DOD Data Oriented Design#^dod-padding]]

В каких сценариях стоит применять DOD? Когда он не оправдан?
?
![[DOD Data Oriented Design#^dod-when-use]]
![[DOD Data Oriented Design#^dod-tradeoff]]

Какой трейдофф у DOD по сравнению с OOD?
?
![[DOD Data Oriented Design#^dod-tradeoff]]

Что выведет этот код — и почему DOD версия будет быстрее на больших данных?
```go
type OOD struct{ X, Y, Z int }
type DOD struct{ X, Y, Z []int }

// OOD: суммируем X у 1M элементов
ood := make([]OOD, 1_000_000)
sum := 0
for _, e := range ood { sum += e.X }

// DOD: суммируем X у 1M элементов
dod := DOD{X: make([]int, 1_000_000)}
sum2 := 0
for _, v := range dod.X { sum2 += v }
```
?
Оба выведут `0` (нулевые значения). Но DOD версия быстрее: `dod.X` — непрерывный срез int, каждая кэшлиния содержит только значения X. В OOD каждая кэшлиния содержит полный OOD-объект (X+Y+Z), из которых 2/3 — «мусор» при суммировании X.
![[DOD Data Oriented Design#^dod-dod-benefit]]
