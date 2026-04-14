#flashcards/str/struktura

Из чего состоит строка в Go на уровне runtime? Сколько байт занимает переменная?
?
![[структура#^str-header-def]]

Почему у строки нет поля capacity (в отличие от slice)?
?
![[структура#^str-no-cap]]

Что происходит с данными при `s2 := s`? Сколько байт копируется?
?
![[структура#^str-assign-share]]

Что выведет этот код?
```go
s := "hello"
s2 := s
s2 = "world"
fmt.Println(s)
```
?
`hello` — `s2 := s` копирует только header (16 байт), данные шарятся. Но `s2 = "world"` перезаписывает header s2, указывая на новые данные. s не затронута.
![[структура#^str-assign-share]]

Что происходит при `s2 := s[1:4]`? Происходит ли копирование данных?
?
![[структура#^str-substring-no-copy]]

Что выведет этот код?
```go
s := "hello"
s2 := s[1:4]
fmt.Println(len(s2), cap(s2))
```
?
Не компилируется — у строки нет cap, `cap(s2)` вернёт ошибку компиляции: `invalid argument: s2 (variable of type string) for cap`.
![[структура#^str-no-cap]]

В чём главный gotcha при работе с подстроками?
?
![[структура#^str-gotcha-memory]]
