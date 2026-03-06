- `go run -race` / `go test -race` — **динамический** анализ на data race. Основан на ThreadSanitizer ^rd-usage
- **Нет false positives**: если нашёл — это точно баг. **Есть false negatives**: видит только выполненные пути ^rd-accuracy
- Overhead: **2-20x** замедление, **5-10x** память. В проде не включать ^rd-overhead

---

## Как работает

```
1. Компилятор добавляет хуки перед каждым доступом к памяти
2. Runtime ведёт shadow memory — история доступов к каждым 8 байтам
3. Каждый доступ: goroutine ID + timestamp (vector clock)
4. Новый доступ → сравнить с историей → есть happens-before?
5. Нет HB + конфликт (read/write или write/write) → DATA RACE
```
^rd-mechanism

**read/read** — не конфликт. Конфликт только если хотя бы одна запись. ^rd-conflict

## Что выводит

```
WARNING: DATA RACE
Write at 0x00c0000b4010 by goroutine 7:
  main.main.func1()
      main.go:12 +0x38

Previous read at 0x00c0000b4010 by main goroutine:
  main.main()
      main.go:14 +0x88

Goroutine 7 (running) created at:
  main.main()
      main.go:11 +0x7c
```
^rd-output

## Ловит ТОЛЬКО data race

| | Ловит? |
|:--|:--|
| Data race | ✅ |
| Race condition | ❌ (логика, не память) |
| Deadlock | ❌ (но runtime сам ловит полный deadlock) |
^rd-scope

## Связь
- [[Data Race]] — что именно ищет детектор
- [[Happens-Before]] — vector clocks проверяют HB
