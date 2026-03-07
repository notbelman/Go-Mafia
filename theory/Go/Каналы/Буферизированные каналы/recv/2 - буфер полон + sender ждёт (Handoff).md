### Recv: буфер полон + sender ждёт (Handoff)

Читаем из буфера и **сразу** кладём данные ждущего sender в освободившуюся ячейку. Паттерн называется **Handoff**. ^recv2-desc

```
ДО:
  буфер: [A, B, C]
  recvx: 0
  sendx: 0
  qcount: 3 (полон)
  sendq: [G1 хочет отправить D] 💤

G2: x := <-ch

  lock()
  x = buf[recvx]         → x = buf[0] → x = A
  recvx++                → recvx = 1
  buf[sendx] = G1.elem   → buf[0] = D
  sendx++                → sendx = 1
  unlock()
  goready(G1)            → G1 в runqueue

ПОСЛЕ:
  буфер: [D, B, C]
  recvx: 1
  sendx: 1
  qcount: 3 (всё ещё полон)
  sendq: пусто
  
  G1: проснулся, его D в буфере ✓
  G2: x = A ✓
```

^recv2-steps

**Ключевой момент**: `qcount` не меняется — буфер остаётся полным. Данные из `sendq` "вливаются" на место только что прочитанного элемента. ^recv2-qcount-unchanged

`goready(G1)` пробуждает спящего sender и помещает его goroutine в runqueue. ^recv2-goready

### Связь
- [[WORK-BASE/interviews/theory/Go/Каналы/Сравнительные таблицы/Внутреннее устройство (hchan)]]
- [[4 - буфер пуст, никто не ждёт]]
