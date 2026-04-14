---
type: task
companies:
  - OZON
topic: PostgreSQL
subtopic:
  - GROUP BY
  - WHERE
  - ORDER BY
title: Подсчёт количества Оскаров у актрис
---

## Условие

Дана таблица "awards" с колонками year, name, award, category.

Написать SQL-запрос, который посчитает сколько "Оскаров" имеют актрисы и выведет результат по убыванию.

## Пример
```
### in
| year | name              | award  | category                |
|------|-------------------|--------|-------------------------|
| 2007 | Jennifer Hudson   | Oscar  | Best Supporting Actress |
| 2009 | Jennifer Hudson   | Grammy | Best Album              |
| 2019 | Lady Gaga         | Oscar  | Best Original Song      |
| 2010 | Lady Gaga         | Grammy | Best Album              |
| 2020 | Lady Gaga         | Grammy | Best Album              |
| 2022 | Lady Gaga         | Grammy | Best Album              |
| 1991 | Barbra Streisand  | Oscar  | Best Original Song      |
| 1964 | Barbra Streisand  | Oscar  | Best Actress            |
| 1986 | Barbra Streisand  | Grammy | Best Album              |

### out
| name              | oscars_count |
|-------------------|--------------|
| Barbra Streisand  | 2            |
| Jennifer Hudson   | 1            |
| Lady Gaga         | 1            |
```

## Решение
```sql
```