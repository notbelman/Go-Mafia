#flashcards/errors/sentinel_vs_custom

Когда выбрать sentinel-ошибку, а когда кастомный тип?
?
![[Sentinel vs кастомный тип vs поведение#^svb-sentinel-when]]
![[Sentinel vs кастомный тип vs поведение#^svb-custom-when]]

Что общего между sentinel-ошибкой и кастомным типом ошибки с точки зрения архитектуры пакетов?
?
![[Sentinel vs кастомный тип vs поведение#^svb-both-coupling]]

Какие поля есть у os.PathError и почему это лучше, чем кодировать их в строку?
?
![[Sentinel vs кастомный тип vs поведение#^svb-custom-example]]

Как проверить ошибку по поведению не импортируя пакет где она определена?
?
![[Sentinel vs кастомный тип vs поведение#^svb-behavior-example]]

Что нужно знать программисту чтобы использовать проверку по поведению? Что ему не нужно?
?
![[Sentinel vs кастомный тип vs поведение#^svb-behavior-example]]

Почему изменение публичной sentinel-ошибки — это breaking change?
?
![[Sentinel vs кастомный тип vs поведение#^svb-api-breaking]]

Что выведет этот код?
```go
type FSError struct{ path string }
func (e *FSError) Error() string { return "fs: " + e.path }
func (e *FSError) Path() string  { return e.path }

err := &FSError{path: "/tmp/x"}
_, ok := err.(interface{ Path() string })
fmt.Println(ok)
```
?
`true` — type assertion к анонимному интерфейсу срабатывает, если у значения есть метод `Path() string`. Импортировать пакет не нужно.
![[Sentinel vs кастомный тип vs поведение#^svb-behavior-example]]
