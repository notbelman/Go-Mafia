#flashcards/unsafe/rules

Сколько легальных паттернов использования unsafe.Pointer определено в документации Go? Что со всем остальным?
?
![[Правила_unsafe_Pointer#^rule1-cast-types]]

Паттерн 1: *T → unsafe.Pointer → *U. Что обязательно для корректности каста?
?
![[Правила_unsafe_Pointer#^rule1-cast-types]]

Паттерн 2: Pointer → uintptr. Для чего это легально и почему обратный каст запрещён?
?
![[Правила_unsafe_Pointer#^rule2-print-only]]

Паттерн 3: арифметика указателей. Почему uintptr → Pointer обязан быть в одном выражении?
?
![[Правила_unsafe_Pointer#^rule3-one-expr]]

Что произойдёт, если сохранить uintptr в переменную перед конверсией обратно в Pointer?
?
![[Правила_unsafe_Pointer#^rule3-one-expr]]

Какая функция появилась в Go 1.17 как безопасная замена ручной арифметике uintptr + Offsetof?
?
![[Правила_unsafe_Pointer#^rule3-unsafe-add]]

Паттерн 4: syscall. Почему конверсию в uintptr нужно делать прямо в аргументе вызова, а не до него?
?
![[Правила_unsafe_Pointer#^rule4-syscall]]

Паттерн 5: reflect.Value.Pointer(). Почему нельзя сохранить результат в переменную перед конверсией в unsafe.Pointer?
?
![[Правила_unsafe_Pointer#^rule5-reflect]]

SliceHeader и StringHeader — deprecated с какой версии Go? Чем их заменить?
?
![[Правила_unsafe_Pointer#^rule6-sliceheader-deprecated]]
![[Правила_unsafe_Pointer#^rule6-new-funcs]]

Какую роль играет go vet при работе с unsafe.Pointer?
?
![[Правила_unsafe_Pointer#^govet-check]]

Что выведет этот код? Правильно ли он написан?
```go
u := uintptr(unsafe.Pointer(&x))
p := unsafe.Pointer(u + 8)
_ = *(*int)(p)
```
?
Код **невалиден** — undefined behavior. GC может переместить `x` между строками 1 и 2. Переменная `u` хранит старый адрес, `p` становится dangling pointer. Правильно: `p := unsafe.Pointer(uintptr(unsafe.Pointer(&x)) + 8)` — одно выражение.
![[Правила_unsafe_Pointer#^rule3-one-expr]]
