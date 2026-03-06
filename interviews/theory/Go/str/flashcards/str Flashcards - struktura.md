#flashcards/str/struktura

Из чего состоит строка в Go на уровне runtime? Сколько байт занимает переменная?
?
![[WORK-BASE/interviews/theory/Go/str/структура#^str-header-def]]

Почему у строки нет поля capacity (в отличие от slice)?
?
![[WORK-BASE/interviews/theory/Go/str/структура#^str-no-cap]]

Что происходит с данными при `s2 := s`? Сколько байт копируется?
?
![[WORK-BASE/interviews/theory/Go/str/структура#^str-assign-share]]

Что выведет этот код?
```go
s := "hello"
s2 := s
s2 = "world"
fmt.Println(s)
```
?
`hello` — `s2 := s` копирует только header (16 байт), данные шарятся. Но `s2 = "world"` перезаписывает header s2, указывая на новые данные. s не затронута.
![[WORK-BASE/interviews/theory/Go/str/структура#^str-assign-share]]

Что происходит при `s2 := s[1:4]`? Происходит ли копирование данных?
?
![[WORK-BASE/interviews/theory/Go/str/структура#^str-substring-no-copy]]

Что выведет этот код?
```go
s := "hello"
s2 := s[1:4]
fmt.Println(len(s2), cap(s2))
```
?
Не компилируется — у строки нет cap, `cap(s2)` вернёт ошибку компиляции: `invalid argument: s2 (variable of type string) for cap`.
![[WORK-BASE/interviews/theory/Go/str/структура#^str-no-cap]]

В чём главный gotcha при работе с подстроками?
?
![[WORK-BASE/interviews/theory/Go/str/структура#^str-gotcha-memory]]
