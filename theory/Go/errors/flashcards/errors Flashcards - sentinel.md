#flashcards/errors/sentinel

Что такое sentinel-ошибка и зачем она нужна?
?
![[Sentinel ошибки#^se-def]]
![[Sentinel ошибки#^se-purpose]]

Назови два классических примера sentinel-ошибок из stdlib Go.
?
![[Sentinel ошибки#^se-purpose]]

Какая проблема у sentinel-ошибок реализованных через var?
?
![[Sentinel ошибки#^se-mutable-example]]

Как сделать ошибку неизменяемой константой в Go? Опиши механизм.
?
![[Sentinel ошибки#^se-const-example]]

Почему в методе Error() у ConstError нужно явное приведение string(e)?
?
![[Sentinel ошибки#^se-const-conversion]]

Работает ли errors.Is с константной ошибкой обёрнутой через fmt.Errorf("%w")?
?
![[Sentinel ошибки#^se-wrap-works]]

Почему нельзя сравнивать sentinel через ==, если ошибка могла быть обёрнута?
?
![[Sentinel ошибки#^se-why-not-eq]]

Что выведет этот код?
```go
type ConstError string
func (e ConstError) Error() string { return string(e) }
const ErrDB ConstError = "db error"

wrapped := fmt.Errorf("op failed: %w", ErrDB)
fmt.Println(wrapped == ErrDB)
fmt.Println(errors.Is(wrapped, ErrDB))
```
?
`false` и `true`. Прямое `==` не раскручивает обёртки, `errors.Is` проходит по цепочке `Unwrap()`.
![[Sentinel ошибки#^se-why-not-eq]]

Что выведет этот код?
```go
type ConstError string
func (e ConstError) Error() string { return string(e) }
const ErrDB ConstError = "db error"

// попытка изменить:
// ErrDB = "hacked"
```
?
Ошибка компиляции — `cannot assign to ErrDB (declared const)`. В отличие от `var`-sentinel, константу изменить нельзя.
![[Sentinel ошибки#^se-const-example]]
