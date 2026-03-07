---
type: task
companies:
  - OZON
topic: PostgreSQL
subtopic:
  - DDL
  -  Проектирование схемы
title: Спроектировать модель библиотеки (автор, книга, читатель)
---

## Условие
Нужно описать модель библиотеки. Есть 3 сущности: "Автор", "Книга", "Читатель".

Физически книга только одна и может быть только у одного читателя. Нужно составить таблицы для библиотеки так, чтобы это учесть.

## Решение

```sql
CREATE TABLE authors (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL
);

CREATE TABLE books (
    id SERIAL PRIMARY KEY,
    title TEXT NOT NULL
);

CREATE TABLE readers (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL
);

-- Книга может быть только у одного читателя (book_id - PRIMARY KEY)
CREATE TABLE book_loans (
    book_id INT NOT NULL REFERENCES books(id) ON DELETE CASCADE,
    reader_id INT NOT NULL REFERENCES readers(id) ON DELETE CASCADE,
    loan_date TIMESTAMP DEFAULT NOW(), 
    PRIMARY KEY (book_id), 
    CONSTRAINT u_book UNIQUE (book_id)
);

-- У книги может быть несколько авторов
CREATE TABLE book_authors (
    book_id INT NOT NULL REFERENCES books(id) ON DELETE CASCADE,
    author_id INT NOT NULL REFERENCES authors(id) ON DELETE CASCADE,
    PRIMARY KEY (book_id, author_id)
);
```

## Запросы к базе

**Название всех книг, которые есть на руках:**
```sql
SELECT b.title
FROM books b
JOIN book_loans bl ON b.id = bl.book_id;
```

**Выбрать название всех книг, у которых больше 3 авторов:**
```sql
SELECT b.title
FROM books b
JOIN book_authors ba ON b.id = ba.book_id
GROUP BY b.title
HAVING COUNT(ba.author_id) > 3;
```
