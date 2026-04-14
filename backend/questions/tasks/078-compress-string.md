---
type: task
companies:
  - VK
topic: Algorithms
subtopic:
  - Strings
  - Run-Length Encoding
title: Сжатие строки по алгоритму Run-Length Encoding
---

## Условие

Дана строка, в ней могут быть идущие подряд символы. Необходимо закодировать её по алгоритму run-length encoding. Если символ встречается один раз, не добавлять число 1, чтобы сжатая строка не была длиннее исходной.

## Пример
```
aaaabb -> a4b2
aaaaa  -> a5
aabbaabb -> a2b2a2b2
abcd -> abcd  (не a1b1c1d1)
```

## Решение
```go
```