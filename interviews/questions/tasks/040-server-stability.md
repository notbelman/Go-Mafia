---
type: task
companies:
  - AVITO
topic: Go
subtopic:
  - Map
  - Slices
title: Распределение серверов по стабильности
---

## Условие

Есть статистика по серверам по стабильности в процентах по бейзлайну 9999.
Необходимо вернуть распределение серверов по показаниям.

## Пример
```
### in
stats = [{server: 1, stability: 99}, {server: 2, stability: 97}, {server: 3, stability: 34}, {server: 4, stability: 97}, {server: 5, stability: 97.1}]

### out
{ 34: [3], 97: [2, 4], 99: [1], 97.1: [5] }
```

## Решение
```go
```