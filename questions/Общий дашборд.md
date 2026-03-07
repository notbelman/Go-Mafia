---
cssclasses: table-max
---
## Алгоритмические задачи
```dataview
TABLE topic, subtopic, title, companies
WHERE contains(type, "tasks")
```
## PostgreSQL Вопросы
```dataview
TABLE companies, subtopic, title
FROM ""
WHERE contains(topic, "PostgreSQL")
SORT choice(type = "question", 0, 1) ASC, length(companies) DESC
```

## Go Вопросы
```dataview
TABLE companies, subtopic, type, title
FROM ""
WHERE contains(topic, "Go")
SORT choice(type = "question", 0, 1) ASC, length(companies) DESC
```
## HR интервью
```dataview
TABLE topic, subtopic, title
WHERE contains(topic, "HR")
```

## OZON
```dataview
TABLE topic, subtopic, type, title
WHERE contains(companies, "OZON")
```

## MTS
```dataview
TABLE topic, subtopic, type, title
WHERE contains(companies, "MTS")
```
## VK
```dataview
TABLE topic, subtopic, type, title
WHERE contains(companies, "VK")
```

## Wildberries
```dataview
TABLE topic, subtopic, type, title
WHERE contains(companies, "Wildberries")
```

## X5
```dataview
TABLE topic, subtopic, type, title
WHERE contains(companies, "X5")
```

## Магнит
```dataview
TABLE topic, subtopic, type, title
WHERE contains(companies, "Магнит")
```

## AVITO
```dataview
TABLE topic, subtopic, type, title
WHERE contains(companies, "AVITO")
```

## Банк Точка
```dataview
TABLE topic, subtopic, type, title
WHERE contains(companies, "Банк Точка")
```

## Ситидрайв
```dataview
TABLE topic, subtopic, type, title
WHERE contains(companies, "Ситидрайв")
```
## Список всех вопросов по популярности
```dataview
TABLE companies, topic, subtopic, title
FROM ""
WHERE type = "question" AND topic != "Rust" AND topic != "HR" AND topic != "Soft"
SORT choice(type = "question", 0, 1) ASC, length(companies) DESC
```

## Все задачки
```dataview
TABLE companies, topic, subtopic, title
FROM ""
WHERE type = "task" AND topic != "Rust"
SORT choice(type = "question", 0, 1) ASC, length(companies) DESC
```
## Все PostgreSQL задачки
```dataview
TABLE companies, topic, subtopic, title
FROM ""
WHERE type = "task" AND topic != "Rust" AND topic = "PostgreSQL"
SORT choice(type = "question", 0, 1) ASC, length(companies) DESC
```
## Все Go задачки
```dataview
TABLE companies, explained, topic, subtopic, title
FROM ""
WHERE type = "task" AND topic != "Rust" AND topic = "Go"
SORT choice(type = "question", 0, 1) ASC, length(companies) DESC
```