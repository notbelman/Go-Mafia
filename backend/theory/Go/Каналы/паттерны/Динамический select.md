- Динамический select — количество case'ов неизвестно на этапе компиляции, определяется в runtime
- Два способа: рекурсивная or-функция (ФП-стиль) или reflect.Select (медленно, но просто)
- Применение: ждать первый из N динамических каналов

---

## Проблема

Обычный select — количество case'ов фиксировано при компиляции: ^dynsel-problem

```go
select {
case <-ch1:  // жёстко прописано
case <-ch2:  // жёстко прописано
}
```

А если каналов 5? 50? Или их количество определяется в runtime?

## Способ 1: рекурсивная or-функция

Ждём первый сработавший из N каналов: ^dynsel-or-def

```go
func or(channels ...<-chan struct{}) <-chan struct{} {
    switch len(channels) {
    case 0:
        return nil  // блокировка навечно
    case 1:
        return channels[0]
    }

    orDone := make(chan struct{})
    go func() {
        defer close(orDone)
        select {
        case <-channels[0]:
        case <-channels[1]:
        case <-or(append(channels[2:], orDone)...):  // рекурсия на хвост
        }
    }()
    return orDone
}
```

**Как работает**: на каждом уровне рекурсии обрабатываем 2 канала + рекурсивно остаток. Когда любой сработает → close(orDone) → раскручивается вверх. ^dynsel-or-how

**Зачем `orDone` в рекурсии**: если сработал канал на верхнем уровне, нужно отменить нижние горутины. `orDone` передаётся вниз как сигнал отмены. ^dynsel-or-done-purpose

## Базовые случаи or()

- `len == 0` → возвращает `nil` (чтение из nil-канала блокирует навечно)
- `len == 1` → возвращает сам канал напрямую ^dynsel-base-cases

## Способ 2: reflect.Select

```go
cases := make([]reflect.SelectCase, len(channels))
for i, ch := range channels {
    cases[i] = reflect.SelectCase{
        Dir:  reflect.SelectRecv,
        Chan: reflect.ValueOf(ch),
    }
}
// можно добавить default:
// cases = append(cases, reflect.SelectCase{Dir: reflect.SelectDefault})

chosen, value, ok := reflect.Select(cases)
fmt.Printf("case %d fired: %v (ok=%v)\n", chosen, value, ok)
```

Рефлексия позволяет и читать, и писать, и закрывать каналы динамически. Но **медленно** — рефлексия в Go дорогая. ^dynsel-reflect

## Сравнение подходов

Рекурсивная or-функция — быстрая, идиоматичная, ФП-стиль, но сложнее в понимании. reflect.Select — медленная, но код проще и позволяет смешивать направления (recv/send/default). ^dynsel-compare

## Когда нужен

- Мониторинг: N сервисов, каждый отдаёт heartbeat-канал, ждём первый сбой
- Таймауты: N задач с разными дедлайнами, реагируем на первый
- Библиотечный код где N неизвестно заранее ^dynsel-when

## Связь
- [[Or-done channel]] — конкретный случай: done + input
- [[Fan-In]] — другой подход к множеству каналов (merge vs first)
