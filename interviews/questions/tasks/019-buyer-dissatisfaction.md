---
type: task
companies:
  - AVITO
topic: Go
subtopic:
  - Binary Search
  -  Sorting
title: Суммарная неудовлетворённость покупателей
explained: "true"
---

## Условие
Есть список товаров и список запросов покупателей. Неудовлетворённость покупателя — минимальная разница между его запросом и ближайшим товаром. Найти суммарную неудовлетворённость всех покупателей.

**Пример:**
- goods = [8, 3, 5]
- buyerNeeds = [5, 6]
- Ответ: 1 (товар 5 точно подходит для первого, для второго ближайший 5, разница 1)

## Решение
Нужно для каждого покупателя найти ближайший по цене товар и посчитать суммарную разницу. Наивный подход — для каждого покупателя пройти все товары — O(N × M). Можно лучше:

**Шаг 1: Сортируем товары** Отсортированный массив позволяет использовать бинарный поиск. O(M log M).

**Шаг 2: Для каждого покупателя — бинарный поиск ближайшего товара** Бинарный поиск находит первый товар ≥ запроса. Ближайший товар — либо он, либо предыдущий (который < запроса). Сравниваем оба и берём минимальную разницу. O(log M) на запрос.

**Шаг 3: Суммируем разницы** Складываем минимальные разницы всех покупателей.

**Сложность:**

- Временная: O(M log M + N log M), где M — товары, N — покупатели
- Пространственная: O(1) дополнительно (сортировка in-place)

```go
package main

import (
	"fmt"
	"math"
	"sort"
)

// Бинарный поиск: находим индекс первого товара >= target
// Если все товары < target, возвращает len(goods)
func binarySearch(goods []int, target int) int {
	l, r := 0, len(goods)
	for l < r {
		m := l + (r-l)/2
		if goods[m] < target {
			l = m + 1
		} else {
			r = m
		}
	}
	return l
}

func total(goods, buyerNeeds []int) int {
	// Шаг 1: сортируем товары для бинарного поиска
	sort.Ints(goods)

	total := 0

	// Шаг 2: для каждого покупателя ищем ближайший товар
	for _, need := range buyerNeeds {
		// idx — первый товар >= need
		idx := binarySearch(goods, need)
		minDiff := math.MaxInt

		// кандидат справа: goods[idx] >= need
		if idx < len(goods) {
			minDiff = goods[idx] - need
		}

		// кандидат слева: goods[idx-1] < need, может быть ближе
		if idx > 0 {
			leftDiff := need - goods[idx-1]
			if leftDiff < minDiff {
				minDiff = leftDiff
			}
		}

		// Шаг 3: накапливаем суммарную неудовлетворённость
		total += minDiff
	}

	return total
}

func main() {
	goods := []int{8, 3, 5}
	buyerNeeds := []int{5, 6}
	result := total(goods, buyerNeeds)
	fmt.Println(result) // 1
}
```