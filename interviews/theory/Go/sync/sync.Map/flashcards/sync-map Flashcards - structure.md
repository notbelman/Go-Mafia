#flashcards/sync-map/structure

Из каких полей состоит sync.Map? Опиши каждое поле и его роль.
?
![[Структура#^struct-definition]]

Какова главная идея архитектуры sync.Map?
?
![[Структура#^struct-main-idea]]

Как связаны read и dirty внутри sync.Map? Что значит "один и тот же entry"?
?
![[Структура#^struct-shared-entry]]

Что произойдёт если обновить entry.p в read — отразится ли это в dirty?
?
![[Структура#^struct-shared-mutation]]

Какой тип имеет поле `read` в sync.Map? Почему именно `atomic.Pointer`, а не просто поле?
?
![[Структура#^struct-definition]]

Что хранит поле `misses` в sync.Map и для чего оно используется?
?
![[Структура#^struct-definition]]

Что такое `readOnly` структура? Какие поля в ней есть и зачем?
?
![[Структура#^struct-definition]]
