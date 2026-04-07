#flashcards/mutex/structure

Что такое sync.Mutex с точки зрения гарантий доступа?
?
![[WORK-BASE/interviews/theory/Go/str/структура#^struct-exclusive]]

Какие два поля у sync.Mutex и за что каждое отвечает?
?
![[WORK-BASE/interviews/theory/Go/str/структура#^struct-fields]]

Нужно ли инициализировать sync.Mutex перед использованием? Почему?
?
![[WORK-BASE/interviews/theory/Go/str/структура#^struct-zero-value]]

Является ли sync.Mutex reentrant? Что будет если одна горутина вызовет Lock() дважды?
?
![[WORK-BASE/interviews/theory/Go/str/структура#^struct-not-reentrant]]

Что произойдёт если вызвать Unlock() без предшествующего Lock()?
?
![[WORK-BASE/interviews/theory/Go/str/структура#^struct-errors]]

Что произойдёт если скопировать sync.Mutex после использования и использовать копию?
?
![[WORK-BASE/interviews/theory/Go/str/структура#^struct-errors]]

Что выведет этот код?
```go
var mu sync.Mutex
mu.Lock()
mu.Lock()
fmt.Println("done")
```
?
Deadlock — горутина пытается захватить мьютекс который уже держит сама. sync.Mutex не reentrant, нет механизма определить что тот же обладатель.
![[WORK-BASE/interviews/theory/Go/str/структура#^struct-not-reentrant]]
