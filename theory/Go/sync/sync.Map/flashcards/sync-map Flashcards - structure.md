#flashcards/sync-map/structure

Из каких полей состоит sync.Map? Опиши каждое поле и его роль.
?
![[WORK-BASE/interviews/theory/Go/str/структура#^struct-definition]]

Какова главная идея архитектуры sync.Map?
?
![[WORK-BASE/interviews/theory/Go/str/структура#^struct-main-idea]]

Как связаны read и dirty внутри sync.Map? Что значит "один и тот же entry"?
?
![[WORK-BASE/interviews/theory/Go/str/структура#^struct-shared-entry]]

Что произойдёт если обновить entry.p в read — отразится ли это в dirty?
?
![[WORK-BASE/interviews/theory/Go/str/структура#^struct-shared-mutation]]

Какой тип имеет поле `read` в sync.Map? Почему именно `atomic.Pointer`, а не просто поле?
?
![[WORK-BASE/interviews/theory/Go/str/структура#^struct-definition]]

Что хранит поле `misses` в sync.Map и для чего оно используется?
?
![[WORK-BASE/interviews/theory/Go/str/структура#^struct-definition]]

Что такое `readOnly` структура? Какие поля в ней есть и зачем?
?
![[WORK-BASE/interviews/theory/Go/str/структура#^struct-definition]]
