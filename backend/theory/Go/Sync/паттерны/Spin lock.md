- Spin lock — мьютекс на цикле CAS: поток крутится в цикле пока не захватит лок
- Busy waiting = активное ожидание: поток жжёт CPU вместо сна. Spin lock — главный пример
- Хорош когда критическая секция короткая (<1мкс): нет overhead на переключение контекста

---

## Идея

Простейший лок — один атомарный флаг:

```go
type SpinLock struct {
    locked atomic.Bool
}

func (s *SpinLock) Lock() {
    for !s.locked.CompareAndSwap(false, true) {
        // busy waiting — крутимся
        runtime.Gosched() // уступаем процессор (опционально)
    }
}

func (s *SpinLock) Unlock() {
    s.locked.Store(false)
}
```

^spinlock-impl

`Lock()`: пытаемся CAS(false→true). Не получилось — пробуем снова. Получилось — мы владелец. ^spinlock-cas-mechanic

## Busy waiting

Поток **не засыпает**, а крутится в цикле проверяя условие. Это и есть **busy waiting** (активное ожидание). Проблема: поток потребляет CPU даже когда ждёт. ^busy-waiting-def

**Когда это OK:**
- критическая секция очень короткая (наносекунды)
- ожидание заведомо короткое
- стоимость park/unpark > стоимости кручения ^busy-waiting-when-ok

**Когда плохо:**
- долгое ожидание — зря жжём CPU
- много потоков на одном ядре — один крутится, другой не может работать ^busy-waiting-when-bad

## runtime.Gosched()

Без `Gosched()` горутина будет крутиться пока не вытеснят (до 10мс). С `Gosched()` — отдаёт управление планировщику, другие горутины получают шанс. ^gosched-effect

## Проблемы spin lock

- **Нет fairness**: кто первый схватил CAS — тот и вошёл. Поток может голодать бесконечно ^spinlock-no-fairness
- **Cache line bouncing**: каждый CAS инвалидирует кэш-линию `locked` на всех ядрах ^spinlock-cache-bouncing
- **Масштабируемость**: чем больше потоков, тем хуже — все молотят по одному адресу ^spinlock-scalability

**Оптимизация — TTAS (Test and Test-And-Set):**

```go
func (s *SpinLock) Lock() {
    for {
        if !s.locked.Load() {           // Test (read-only, из кэша)
            if s.locked.CompareAndSwap(false, true) { // Test-And-Set
                return
            }
        }
        runtime.Gosched()
    }
}
```

Сначала `Load()` (дешёвый read), потом CAS только если флаг свободен. Меньше инвалидаций кэша. ^ttas-explanation

## Связь
- [[Ticket lock]] — решает проблему fairness
- [[CAS паттерны]] — CAS как основа спинлока
- [[True sharing и False sharing]] — cache line bouncing при спинлоке
- [[Примитивный мьютекс]] — альтернатива: park вместо spinning
