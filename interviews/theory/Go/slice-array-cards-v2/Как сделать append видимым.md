[[slice Flashcards - append_visible]]
### Способ 1: вернуть slice (идиоматический)

```go
func appendAndReturn(s []int) []int {
    return append(s, 4)
}

func main() {
    s := []int{1, 2, 3}
    s = appendAndReturn(s)  // ← присваиваем результат
    fmt.Println(s)          // [1 2 3 4]
}
```

Идиоматический способ — возврат нового slice из функции. Вызывающий код присваивает результат. ^append-visible-return

### Способ 2: передать указатель на slice

```go
func appendPtr(s *[]int) {
    *s = append(*s, 4)
}

func main() {
    s := []int{1, 2, 3}
    appendPtr(&s)
    fmt.Println(s)  // [1 2 3 4]
}
```

Второй способ — передать `*[]int`. Функция разыменовывает указатель и присваивает результат append напрямую в оригинальный header. ^append-visible-pointer

Способ 1 идиоматичнее в Go — передача указателя на slice используется редко и усложняет сигнатуру. ^append-visible-tradeoff

## Связь
- [[append внутри функции — НЕ видно снаружи]] — почему append не виден
- [[append]] — почему append возвращает slice
