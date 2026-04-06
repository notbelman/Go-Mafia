- **идея:** одинаковые строки → один экземпляр в памяти, все ссылаются на него
- Go **автоматически** интернирует строковые литералы (в text segment, read-only)
- `var s = "test"` и `const c = "test"` → **одинаковый адрес** (литерал = де-факто константа)
- runtime строки — руками через `map[string]string` или пакет `unique` (Go 1.23)
- `unique.Make` → Handle с указателем, сравнение Handle за **O(1)** вместо O(n)

---

## Проблема без interning

```go
// "the" встречается 5000 раз в файле
words[142]  ──► ['t','h','e']   аллокация 1
words[517]  ──► ['t','h','e']   аллокация 2  
words[891]  ──► ['t','h','e']   аллокация 3
... ещё 4997 копий
```

Без интернирования каждое вхождение одной строки создаёт отдельную аллокацию в heap — даже если строки идентичны по содержимому. ^interning-problem

## Ручной interning через map

```go
pool := map[string]string{}

func intern(s string) string {
    if existing, ok := pool[s]; ok {
        return existing
    }
    pool[s] = s
    return s
}
// "the" хранится 1 раз, все 5000 указывают на неё
```

Ручной пул: первое обращение кладёт строку в map, последующие возвращают уже сохранённый экземпляр. Все 5000 копий указывают на один и тот же backing array. ^interning-manual-map

## Интернирование констант компилятором

```go
s1 := "hello"
s2 := "hello"
const c = "hello"

fmt.Println(unsafe.StringData(s1) == unsafe.StringData(s2))  // true — один адрес!
fmt.Println(unsafe.StringData(s1) == unsafe.StringData(c))   // true
```

Все три живут в **text segment** (read-only). Строковый литерал де-факто является константой, потому что строки в Go immutable. ^interning-compiler-text-segment

## Пакет unique (Go 1.23)

```go
h1 := unique.Make("hello" + "world")  // некоторая конкатенация
h2 := unique.Make("hello" + "world")  // та же строка

h1 == h2  // true — O(1), сравнение указателей
```

Внутри: глобальный пул, отдельная map для каждого типа, goroutine-safe. Работает не только со строками — любой `comparable` тип. ^interning-unique-internals

Бенчмарк: сравнение через Handle в разы быстрее лексикографического сравнения длинных строк. ^interning-unique-bench

## Когда полезно

Парсинг данных с повторами: логи, CSV, JSON с одинаковыми ключами, дедупликация больших структур. ^interning-use-cases

## Связь
- [[иммутабельность]] — литералы в text segment благодаря иммутабельности
- [[Сравнение строк]] — interning ускоряет сравнение до O(1)
- [[Unsafe string to byte]] — попытка изменить литерал в text segment = crash
