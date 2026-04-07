**Проблема:** данные в двух местах + удалённые записи ^len-problem

```
read:  {a: value, b: nil, c: expunged}  // 1 живой
dirty: {a: value, d: value}              // 2 живых, но a — тот же
```

Сколько элементов? Надо:
1. Взять лок
2. Обойти ОБЕ map
3. Отфильтровать nil и expunged
4. Убрать дубликаты

= O(n), блокирует map, противоречит философии lock-free чтений. ^len-complexity

**Workaround:** ^len-workaround

```go
count := 0
m.Range(func(_, _ any) bool {
    count++
    return true
})
```

Но это тоже O(n) и может триггернуть promotion. ^len-workaround-detail
