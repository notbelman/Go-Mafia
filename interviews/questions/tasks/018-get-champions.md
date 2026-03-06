---
type: task
companies:
  - AVITO
topic: Go
subtopic:
  - Map
  - Slices
  - Algorithms
title: Определить чемпионов по шагам за все дни соревнований
explained: "true"
---
## Условие
Определить чемпионов по шагам. Чемпион — пользователь, который:
1. Участвовал во всех днях соревнований
2. Прошёл наибольшее количество шагов суммарно

## Решение
Нам нужно найти пользователей, которые участвовали каждый день И набрали максимум шагов суммарно. Делаем это за один проход по данным + два прохода по пользователям:

**Шаг 1: Собираем статистику по каждому пользователю** Проходим по всем дням и записям. Для каждого пользователя накапливаем две вещи: сколько дней он участвовал и сколько шагов прошёл суммарно. Храним в `map[int]MetaUser`.

**Шаг 2: Находим максимум шагов среди «полных» участников** Проходим по map, смотрим только тех, у кого `Days == dayCount` (участвовал каждый день). Среди них ищем максимальную сумму шагов.

**Шаг 3: Собираем всех чемпионов** Ещё раз проходим по map, берём всех с `Days == dayCount` и `Steps == maxSteps`. Чемпионов может быть несколько.

**Сложность:**

- Временная: O(N + U), где N — общее количество записей, U — уникальные пользователи
- Пространственная: O(U) — map с метаданными

```go
package main

import "fmt"

type Statistic struct {
	UserID int
	Steps  int
}

type Result struct {
	UserIDs []int
	Steps   int
}

type MetaUser struct {
	Steps int
	Days  int
}

func getChampions(statistics [][]Statistic) Result {
	var result Result
	var maxStepCount int
	users := make(map[int]MetaUser)
	dayCount := len(statistics)

	if dayCount == 0 {
		return result
	}

	// Шаг 1: собираем дни и шаги по каждому пользователю
	for _, day := range statistics {
		for _, userStat := range day {
			u := users[userStat.UserID]
			u.Days++
			u.Steps += userStat.Steps
			users[userStat.UserID] = u
		}
	}

	// Шаг 2: максимум шагов среди участвовавших каждый день
	for _, u := range users {
		if u.Days == dayCount && u.Steps > maxStepCount {
			maxStepCount = u.Steps
		}
	}

	// Шаг 3: все пользователи с максимумом — чемпионы
	for id, u := range users {
		if u.Days == dayCount && u.Steps == maxStepCount {
			result.UserIDs = append(result.UserIDs, id)
		}
	}

	result.Steps = maxStepCount
	return result
}

func main() {
	statistics := [][]Statistic{
		{{UserID: 1, Steps: 1000}, {UserID: 2, Steps: 1500}},
		{{UserID: 2, Steps: 1000}},
	}
	result := getChampions(statistics)
	fmt.Println(result) // {[2] 2500}
}
```