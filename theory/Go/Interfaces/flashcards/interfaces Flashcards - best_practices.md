#flashcards/interfaces/best_practices

Почему маленькие интерфейсы лучше больших?
?
![[Best practices#^bp-small-iface]]

В чём смысл правила "accept interfaces, return structs"?
?
![[Best practices#^bp-accept-return]]

Почему функция должна принимать интерфейс, а не конкретный тип?
?
![[Best practices#^bp-accept-why]]

Почему функция должна возвращать конкретный тип, а не интерфейс?
?
![[Best practices#^bp-return-struct-why]]

Когда оправдано возвращать интерфейс вместо конкретного типа?
?
![[Best practices#^bp-return-iface-exception]]

Какую compile-time гарантию даёт возврат интерфейса?
?
![[Best practices#^bp-return-iface-compile]]

Что такое interface pollution и в каких трёх случаях интерфейс оправдан?
?
![[Best practices#^bp-pollution]]
![[Best practices#^bp-when-iface-needed]]

Процитируй принцип Rob Pike про интерфейсы — в чём суть?
?
![[Best practices#^bp-pike-quote]]

Чем подход Go к интерфейсам отличается от Java/C++?
?
![[Best practices#^bp-java-approach]]
![[Best practices#^bp-go-approach]]

Что выведет этот код — и нарушает ли он best practices?
```go
type UserGetter interface {
    GetUser(id int) string
}
type userService struct{}
func (u *userService) GetUser(id int) string { return "user" }

func process(g UserGetter) string {
    return g.GetUser(1)
}
func main() {
    s := &userService{}
    fmt.Println(process(s))
}
```
?
`user` — код работает корректно. Но интерфейс `UserGetter` избыточен если `userService` — единственная реализация и нет тестов/межпакетных зависимостей. Нарушение: interface pollution.
![[Best practices#^bp-pollution]]

Что выведет этот код и какой best practice он демонстрирует?
```go
func NewReader() io.Reader {
    return &strings.Reader{}
}
func main() {
    r := NewReader()
    fmt.Printf("%T\n", r)
}
```
?
`*strings.Reader` — возврат интерфейса даёт compile-time гарантию что тип реализует контракт. Исключение из "return structs".
![[Best practices#^bp-return-iface-compile]]
