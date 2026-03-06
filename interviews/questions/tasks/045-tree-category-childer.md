---
type: task
companies:
  - AVITO
topic: Go
subtopic:
  - Tree
  - DFS
  - Recursion
title: Поиск подкатегории в дереве и вывод пути
---

## Условие

Есть дерево категорий товаров, размещаемых на Авито. Нужно найти заданную подкатегорию и вывести путь до неё от корневой категории, если она присутствует в дереве.

## Пример
```
### in
class Category {
    name: string;
    children: Array<Category>;
}

root = {
    name: "root",
    children: [
        {
            name: "Бытовая техника",
            children: [
                {
                    name: "Телевизоры",
                    children: [
                        { name: "ЭЛТ", children: [] },
                        { name: "LED", children: [] },
                        { name: "OLED", children: [] }
                    ]
                },
                {
                    name: "Холодильники",
                    children: [
                        { name: "Двухкамерные", children: [] },
                        { name: "Однокамерные", children: [] }
                    ]
                },
                {
                    name: "Утюги",
                    children: []
                }
            ]
        },
        {
            name: "Растения",
            children: [
                { name: "Комнатные", children: [] },
                { name: "Садовые", children: [] }
            ]
        }
    ]
}

search_name = "OLED"

### out
Бытовая техника > Телевизоры > OLED
```

## Решение
```go
```