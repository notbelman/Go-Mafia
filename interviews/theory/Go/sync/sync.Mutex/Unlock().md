```mermaid
flowchart TD
    A[Unlock] --> B[Fast Path:<br>new = atomic.Add state,-1]
    B --> C{new == 0?}
    
    C -->|YES| D[return<br>никто не ждёт]
    C -->|NO| E[unlockSlow]
    
    E --> F{Starvation mode?}
    
    F -->|NO<br>Normal| G{waiters==0 OR<br>locked OR woken OR<br>starving?}
    F -->|YES<br>Starvation| H[runtime_Semrelease<br>handoff=true]
    
    G -->|YES| I[return<br>ничего не делаем]
    G -->|NO| J[state: waiters--<br>woken = 1]
    
    J --> K[runtime_Semrelease<br>handoff=false]
    K --> L[Будим одного<br>Он конкурирует с новыми]
    
    H --> M[Лок передаётся напрямую<br>первому в очереди FIFO<br>Никакой конкуренции]
```

## Описание алгоритма

**Fast path:** `atomic.Add(state, -mutexLocked)`. Если результат == 0 — нет ожидающих, выходим. ^unlock-fast-path

**unlockSlow, Normal mode:** проверяем нужно ли будить кого-то. Условия при которых ничего не делаем: нет ожидающих (waiters==0), или лок уже снова захвачен (locked), или уже есть разбуженный (woken), или режим starving. ^unlock-slow-normal-skip

Если нужно будить: `waiters--`, устанавливаем `woken=1`, вызываем `runtime_Semrelease(handoff=false)` — горутина проснётся и будет **конкурировать** с новыми горутинами. ^unlock-slow-normal-wake

**unlockSlow, Starvation mode:** `runtime_Semrelease(handoff=true)` — лок передаётся **напрямую** первому в очереди FIFO, без конкуренции. ^unlock-starvation-handoff
