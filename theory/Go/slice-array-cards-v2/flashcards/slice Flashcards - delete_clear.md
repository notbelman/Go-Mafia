#flashcards/slice/delete_clear

Какая сложность удаления из конца, начала, середины slice?
?
![[Удаление и очистка slice#^delete-end]]
![[Удаление и очистка slice#^delete-front]]
![[Удаление и очистка slice#^delete-middle]]

Почему удаление из конца `s[:len(s)-1]` может привести к утечке памяти?
?
![[Удаление и очистка slice#^delete-end-memory-leak]]

Почему многократное `s = s[1:]` приводит к реаллокации?
?
![[Удаление и очистка slice#^delete-front-cap]]

С какой версии Go доступен `slices.Delete`?
?
![[Удаление и очистка slice#^delete-middle-go121]]

Как удалить элемент из середины slice за O(1) если порядок неважен?
?
![[Удаление и очистка slice#^delete-swap-trick]]

Как реализовать стек (push/pop) на slice?
?
![[Удаление и очистка slice#^stack-queue]]

Почему slice — плохой выбор для очереди с pop front?
?
![[Удаление и очистка slice#^queue-circular-buffer]]

В чём разница между `s = nil` и `s = s[:0]` для очистки slice?
?
![[Удаление и очистка slice#^clear-nil-vs-zero]]

Что делает `clear(s)`? Как меняются len и cap?
?
![[Удаление и очистка slice#^clear-builtin]]

С какой версии Go доступен `clear()`?
?
![[Удаление и очистка slice#^clear-builtin]]

Перечисли 4 способа очистить slice и опиши что происходит с памятью в каждом случае.
?
![[Удаление и очистка slice#^clear-four-ways]]

Почему `for i := range s { s[i] = 0 }` быстрее чем `for i := range s { s[i] = 5 }`?
?
![[Удаление и очистка slice#^memclr-detail]]

Что такое memclr оптимизация в Go и какие технологии она использует?
?
![[Удаление и очистка slice#^memclr-detail]]

Что выведет этот код?
```go
s := []int{1, 2, 3, 4, 5}
s = s[:len(s)-1]
fmt.Println(len(s), cap(s))
```
?
`4 5` — удаление из конца уменьшает len на 1, cap не меняется. Элемент `5` остаётся в underlying array.
![[Удаление и очистка slice#^delete-end]]

Что выведет этот код?
```go
s := []int{1, 2, 3, 4, 5}
s = s[1:]
fmt.Println(len(s), cap(s))
s = s[1:]
fmt.Println(len(s), cap(s))
```
?
`4 4` / `3 3` — каждый `s[1:]` уменьшает и len, и cap на 1 (ptr сдвигается вперёд).
![[Удаление и очистка slice#^delete-front-cap]]

Что выведет этот код?
```go
s := []int{1, 2, 3}
s = s[:0]
fmt.Println(len(s), cap(s), s == nil)
```
?
`0 3 false` — `s[:0]` сбрасывает len в 0, cap сохранён (3), ptr не nil, поэтому `s == nil` вернёт false.
![[Удаление и очистка slice#^clear-nil-vs-zero]]

Что выведет этот код?
```go
s := []int{1, 2, 3}
s2 := s
s = nil
fmt.Println(s2)
```
?
`[1 2 3]` — `s = nil` меняет только переменную `s` (её header), underlying array не затронут. `s2` по-прежнему ссылается на тот же массив.
![[Удаление и очистка slice#^clear-nil-vs-zero]]
