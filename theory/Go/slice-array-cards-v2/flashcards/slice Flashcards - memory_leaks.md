#flashcards/slice/memory_leaks

Почему подсрез от большого слайса вызывает утечку памяти?
?
Подсрез делит underlying array с оригиналом. Пока жив хотя бы один слайс, указывающий на array — GC не может его собрать. 20 байт подсреза держат 1 ГБ оригинала.
![[Slice утечки памяти#^leak-subslice-mechanism]]

Почему возврат `&data[i]` из функции — утечка?
?
Указатель ссылается внутрь underlying array. Один `*int` держит весь массив живым — GC не соберёт его, пока указатель существует.
![[Slice утечки памяти#^leak-ptr-why]]

Почему `slices.Clip` не решает утечку?
?
`Clip` только уменьшает `cap` в header (через three-index slice), но `ptr` всё ещё указывает в старый array. Память не освобождается.
![[Slice утечки памяти#^leak-clip-why]]

Как правильно исправить утечку подсреза?
?
Перекопировать в новый слайс: `make + copy` или `slices.Clone`. Новый underlying array — GC забирает старый.
![[Slice утечки памяти#^leak-clone-fix]]

Что выведет этот код и почему?
```go
func leak() []byte {
    big := make([]byte, 1<<30) // 1 ГБ
    big[0] = 1
    return big[:1]
}
func main() {
    s := leak()
    runtime.GC()
    fmt.Println(len(s), cap(s))
}
```
?
`1 1073741824` — `s` это подсрез, его `ptr` указывает в тот же 1 ГБ array. `runtime.GC()` не поможет — ссылка жива.
![[Slice утечки памяти#^leak-subslice-mechanism]]
![[Slice утечки памяти#^leak-gc-useless]]