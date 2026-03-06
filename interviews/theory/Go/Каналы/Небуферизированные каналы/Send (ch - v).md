```
ch == nil?     → блокировка навечно
ch closed?     → panic

lock()

┌─────────────────────────────────────────────────
│ Receiver ждёт в recvq?
└─────────────────────────────────────────────────

  ДА → Direct Copy sendDirect()

      ДО:
        recvq: [G1 ждёт получить в &x] 💤
        sendq: пусто

      ЧТО ДЕЛАЕМ:
        G1.elem = v   → копируем v в стек G1 (в его x)
        unlock()
        goready(G1)   → G1 в runqueue

      ПОСЛЕ:
        recvq: пусто
        G1: runnable → running ✓
        G1.x = v ✓

  НЕТ → засыпаем

      ДО:
        recvq: пусто
        sendq: пусто

      ЧТО ДЕЛАЕМ:
        создаём sudog (elem = &v)
        встаём в sendq
        unlock()
        gopark()   → running → waiting 💤

        ... кто-то делает receive ...
        ... он копирует v из нашего стека ...
        ... он вызывает goready(нас) ...

        просыпаемся, v уже отправлен ✓

      ПОСЛЕ:
        sendq: пусто
        отправили ✓
```

**Ветка "nil-канал":** send в `nil`-канал блокирует горутину навечно. ^send-nil

**Ветка "closed-канал":** send в закрытый канал вызывает **panic** (в отличие от receive, который возвращает zero value). ^send-closed-panic

**Ветка "Direct Copy" (receiver ждёт в recvq):** значение `v` копируется напрямую в стек ожидающего получателя (`G1.elem = v`), затем `goready(G1)`. Данные никогда не касаются буфера канала. ^send-direct-copy

**Ветка "засыпаем" (recvq пуст):** создаётся `sudog` с указателем на значение отправителя (`elem = &v`), горутина встаёт в `sendq` и уходит в `gopark`. Когда придёт получатель — он заберёт данные прямо из стека отправителя. ^send-sleep

**Порядок проверок:** nil → closed → lock → recvq? → Direct Copy или sleep. ^send-order

**Асимметрия поведения closed-канала:** send → panic, receive → zero value + false. Это принципиальное различие важно для паттернов с закрытием канала как сигнала. ^send-vs-recv-closed
