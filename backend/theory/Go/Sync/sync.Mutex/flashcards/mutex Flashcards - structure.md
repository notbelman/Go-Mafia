#flashcards/mutex/structure

Что такое sync.Mutex с точки зрения гарантий доступа?
?
![[sync.Mutex/Структура#^struct-exclusive]]

Какие два поля у sync.Mutex и за что каждое отвечает?
?
![[sync.Mutex/Структура#^struct-fields]]

Нужно ли инициализировать sync.Mutex перед использованием? Почему?
?
![[sync.Mutex/Структура#^struct-zero-value]]

Является ли sync.Mutex reentrant? Что будет если одна горутина вызовет Lock() дважды?
?
![[sync.Mutex/Структура#^struct-not-reentrant]]

Что произойдёт если вызвать Unlock() без предшествующего Lock()?
?
![[sync.Mutex/Структура#^struct-errors]]

Что произойдёт если скопировать sync.Mutex после использования и использовать копию?
?
![[sync.Mutex/Структура#^struct-errors]]

Что выведет этот код?
```go
var mu sync.Mutex
mu.Lock()
mu.Lock()
fmt.Println("done")
```
?
Deadlock — горутина пытается захватить мьютекс который уже держит сама. sync.Mutex не reentrant, нет механизма определить что тот же обладатель.
![[sync.Mutex/Структура#^struct-not-reentrant]]
