### Recv: буфер пуст + sender ждёт (Direct Copy / Bypass)

Берём данные **напрямую из стека sender**, минуя буфер. Паттерн называется **Direct Copy Bypass**. ^recv3-desc

```
ДО:
  буфер: [_, _, _]
  qcount: 0
  sendq: [G1 хочет отправить D] 💤

G2: x := <-ch

  lock()
  x = G1.elem   → x = D (копируем из стека G1)
  unlock()
  goready(G1)   → G1 в runqueue

ПОСЛЕ:
  буфер: [_, _, _] (не трогали)
  sendq: пусто
  
  G1: проснулся, его D отдан ✓
  G2: x = D ✓
```

^recv3-steps

**Оптимизация**: буфер полностью обходится — одна копия вместо двух (стек→буфер→стек). ^recv3-optimization

Это возможно потому что sender уже заблокирован и держит `elem` в своём sudog — receiver берёт данные оттуда напрямую. ^recv3-sudog

### Связь
- [[WORK-BASE/interviews/theory/Go/Каналы/Сравнительные таблицы/Внутреннее устройство (hchan)]]
- [[2 - буфер пуст + receiver ждёт (Direct Copy)]]
