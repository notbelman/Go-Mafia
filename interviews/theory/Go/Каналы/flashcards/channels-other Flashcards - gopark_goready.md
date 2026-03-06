#flashcards/channels-other/gopark_goready

Что делают `gopark()` и `goready()`? Какие переходы состояний они вызывают?
?
![[gopark() и goready()#^gopark-def]]
![[gopark() и goready()#^goready-def]]

Почему `goready()` всегда нужно вызывать ПОСЛЕ `unlock()`, а не до?
?
![[gopark() и goready()#^goready-why-after-unlock]]

Что выведет этот код (оцени порядок вызовов в runtime)?
```
// Правильный вариант:
unlock()
goready(G1)

// Неправильный вариант:
goready(G1)
unlock()
```
Какая разница в поведении?
?
Правильный: G1 просыпается, мьютекс уже свободен — нет лишних переключений. Неправильный: G1 немедленно пытается захватить мьютекс, видит что он занят, встаёт в ожидание — лишние переключения контекста и задержки.
![[gopark() и goready()#^goready-order-rule]]
![[gopark() и goready()#^goready-why-after-unlock]]
