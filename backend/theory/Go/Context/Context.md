## Context в Go

#### [[Что такое context]]
Что такое context.Context, зачем нужен, дерево контекстов

#### [[Context ошибки и правила]]
Правила: не хранить в struct, первый параметр, не nil, не передавать данные

#### [[context Background и TODO]]
context.Background() vs context.TODO() — когда что использовать

---

### Создание контекстов

#### [[context WithCancel]]
WithCancel — отмена вручную, cancel функция, утечки если не вызвать cancel

#### [[context WithTimeout]]
WithTimeout — автоотмена через duration, deadline под капотом

#### [[context WithDeadline]]
WithDeadline — отмена в конкретный момент времени

#### [[context WithValue]]
WithValue — передача значений, типобезопасные ключи, не для параметров

#### [[context WithoutCancel]]
WithoutCancel (Go 1.21) — дочерний контекст без наследования отмены

#### [[context AfterFunc]]
context.AfterFunc (Go 1.21) — callback при отмене контекста

---

### Паттерны

#### [[errgroup с контекстами]]
errgroup.WithContext — группа горутин с общим контекстом и первой ошибкой

#### [[Оборачивание функций без контекста]]
Как добавить контекст к функции которая его не принимает
