#flashcards/sync-map/structure

Из каких полей состоит sync.Map? Опиши каждое поле и его роль.
?
![[sync.Map/Структура#^struct-definition]]

Какова главная идея архитектуры sync.Map?
?
![[sync.Map/Структура#^struct-main-idea]]

Как связаны read и dirty внутри sync.Map? Что значит "один и тот же entry"?
?
![[sync.Map/Структура#^struct-shared-entry]]

Что произойдёт если обновить entry.p в read — отразится ли это в dirty?
?
![[sync.Map/Структура#^struct-shared-mutation]]

Какой тип имеет поле `read` в sync.Map? Почему именно `atomic.Pointer`, а не просто поле?
?
![[sync.Map/Структура#^struct-definition]]

Что хранит поле `misses` в sync.Map и для чего оно используется?
?
![[sync.Map/Структура#^struct-definition]]

Что такое `readOnly` структура? Какие поля в ней есть и зачем?
?
![[sync.Map/Структура#^struct-definition]]
