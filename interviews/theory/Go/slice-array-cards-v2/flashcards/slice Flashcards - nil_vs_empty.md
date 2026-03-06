#flashcards/slice/nil_vs_empty

Чем nil slice отличается от empty slice при сравнении с nil?
?
![[nil slice vs empty slice#^nil-vs-empty-nil-def]]
![[nil slice vs empty slice#^nil-vs-empty-empty-def]]

Каковы len и cap у nil slice и empty slice?
?
![[nil slice vs empty slice#^nil-vs-empty-len-cap]]

Почему стоит проверять `len(s) == 0` вместо `s == nil`?
?
![[nil slice vs empty slice#^nil-vs-empty-check]]

Что вернёт `reflect.DeepEqual(nilSlice, emptySlice)` где `var nilSlice []int` и `emptySlice := []int{}`?
?
![[nil slice vs empty slice#^nil-vs-empty-deepequal]]

Чем отличается underlying array у nil slice и empty slice?
?
![[nil slice vs empty slice#^nil-vs-empty-underlying]]

Как ведёт себя nil slice при JSON-сериализации? А empty slice?
?
![[nil slice vs empty slice#^nil-vs-empty-json]]

Почему `reflect.DeepEqual` ломает тесты при сравнении nil и empty slice?
?
![[nil slice vs empty slice#^nil-vs-empty-deepequal-detail]]

Что рекомендует Go Wiki: nil slice или empty slice? Почему?
?
![[nil slice vs empty slice#^nil-vs-empty-wiki]]

Когда оправдано использовать empty slice вместо nil slice?
?
![[nil slice vs empty slice#^nil-vs-empty-exception]]

Что выведет этот код?
```go
var s []int
fmt.Println(s == nil, len(s), cap(s))
s = append(s, 1)
fmt.Println(s == nil, len(s))
```
?
`true 0 0` — nil slice имеет len=cap=0 и равен nil. После append: `false 1` — append создал новый underlying array, s больше не nil.
![[nil slice vs empty slice#^nil-vs-empty-nil-def]]
![[nil slice vs empty slice#^nil-vs-empty-len-cap]]

Что выведет этот код?
```go
type Resp struct {
    Items []string `json:"items"`
}
r1 := Resp{}
r2 := Resp{Items: []string{}}
b1, _ := json.Marshal(r1)
b2, _ := json.Marshal(r2)
fmt.Println(string(b1))
fmt.Println(string(b2))
```
?
`{"items":null}` и `{"items":[]}` — нулевое значение поля — nil slice, сериализуется как null. Явно инициализированный empty slice — как [].
![[nil slice vs empty slice#^nil-vs-empty-json]]
