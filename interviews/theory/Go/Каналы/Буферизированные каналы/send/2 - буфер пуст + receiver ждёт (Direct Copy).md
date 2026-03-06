### Send: буфер пуст + receiver ждёт (Direct Copy)

Отдаём данные **напрямую в стек receiver**, минуя буфер. Паттерн называется **Direct Copy**. ^send2-desc

```
ДО:
  буфер: [_, _, _]
  qcount: 0
  recvq: [G1 ждёт получить в &x] 💤

G2: ch <- D

  lock()
  G1.elem = D   → копируем D в стек G1 (в его x)
  unlock()
  goready(G1)   → G1 в runqueue

ПОСЛЕ:
  буфер: [_, _, _] (не трогали)
  recvq: пусто
  
  G1: проснулся, x = D ✓
  G2: отправил ✓
```

^send2-steps

**Оптимизация**: буфер обходится полностью. Данные копируются напрямую из стека sender в стек receiver через sudog. ^send2-optimization

Sender пишет в `G1.elem` — это указатель на переменную x в стеке спящего G1. ^send2-elem-ptr

### Связь
- [[WORK-BASE/interviews/theory/Go/Каналы/Сравнительные таблицы/Внутреннее устройство (hchan)]]
- [[4 - буфер пуст, никто не ждёт]]
