```
ch == nil?          → блокировка навечно
ch closed?          → return zero value, false

lock()

┌─────────────────────────────────────────────────
│ Sender ждёт в sendq?
└─────────────────────────────────────────────────

  ДА → Direct Copy sendDirect()

      ДО:
        sendq: [G1 хочет отправить D] 💤
        recvq: пусто

      ЧТО ДЕЛАЕМ:
        x = G1.elem   → x = D (копируем из стека G1)
        unlock()
        goready(G1)   → G1 в runqueue

      ПОСЛЕ:
        sendq: пусто
        G1: runnable → running ✓
        x = D ✓

  НЕТ → засыпаем

      ДО:
        sendq: пусто
        recvq: пусто

      ЧТО ДЕЛАЕМ:
        создаём sudog (elem = &x)
        встаём в recvq
        unlock()
        gopark()   → running → waiting 💤

        ... кто-то делает send ...
        ... он копирует данные в наш стек ...
        ... он вызывает goready(нас) ...

        просыпаемся, x уже заполнен ✓

      ПОСЛЕ:
        recvq: пусто
        x = значение от sender ✓
```

**Ветка "nil-канал":** receive из `nil`-канала блокирует горутину навечно (gopark без возможности пробуждения). ^recv-nil

**Ветка "closed-канал":** receive из закрытого канала немедленно возвращает zero value и `false` без блокировки. ^recv-closed

**Ветка "Direct Copy" (sender ждёт в sendq):** данные копируются напрямую из стека заблокированного отправителя (`G1.elem → x`), затем `goready(G1)` — отправитель пробуждается. Буфер не используется. ^recv-direct-copy

**Ветка "засыпаем" (sendq пуст):** создаётся `sudog` с указателем на переменную получателя (`elem = &x`), горутина встаёт в `recvq` и уходит в `gopark`. Когда придёт отправитель — он запишет данные прямо в стек получателя и вызовет `goready`. ^recv-sleep

**Порядок проверок:** nil → closed → lock → sendq? → Direct Copy или sleep. ^recv-order
