```mermaid
flowchart TD
    A[Lock] --> B[Fast Path: CAS 0→1]
    
    B -->|success| C[return]
    B -->|fail| D[Slow Path]
    
    D --> E{Starvation mode?}
    
    E -->|YES| F[waiters++<br>В очередь, спать]
    E -->|NO| G{Можно спиннить?<br>многоядерка +<br>idle P + iter < 4}
    
    G -->|YES| H[Spin ~30 циклов<br>x 4 итерации]
    G -->|NO| K
    
    H -->|CAS ok| C
    H -->|CAS fail| I{iter++ < 4?}
    
    I -->|YES| H
    I -->|NO| K[waiters++<br>В очередь, спать]
    
    F --> L1[Проснулся<br>Лок уже твой handoff]
    K --> L2[Проснулся<br>Unlock сигнал]
    
    L2 --> M{Ждали > 1ms?}
    
    M -->|YES| N[starving = true]
    M -->|NO| O[остаёмся в normal]
    
    N --> P[Пробуем CAS<br>конкурируем с новыми]
    O --> P
    
    L1 --> Q{Последний в очереди<br>ИЛИ ждал < 1ms?}
    Q -->|YES| R[starving = false<br>Выход из starvation]
    Q -->|NO| S[Остаёмся в starvation]
    
    R --> C
    S --> C
    P -->|success| C
    P -->|fail| T[В начало очереди<br>спать]
    T --> L2
```

## Описание алгоритма

**Fast path:** одна атомарная операция CAS 0→1. Если мьютекс свободен — захватываем мгновенно, без slow path. ^lock-fast-path

**Slow path, Normal mode, условия для спиннинга:** многоядерность + есть idle P + меньше 4 итераций пройдено. Каждая итерация — ~30 циклов процессора. Максимум 4 итерации (120 циклов суммарно). ^lock-spin-conditions

Если спиннинг не помог — `waiters++` и горутина засыпает через `runtime_SemacquireMutex`. ^lock-sleep

**Slow path, Starvation mode:** спиннинг пропускается сразу, горутина идёт в очередь — handoff. После пробуждения лок уже у неё, конкуренции нет. ^lock-starvation-path

**После пробуждения в Normal mode:** если горутина ждала > 1ms — устанавливает флаг `starving`, конкурирует за CAS. При проигрыше встаёт в **начало** очереди (не в конец). ^lock-wakeup-normal

**Выход из Starvation mode (после получения лока через handoff):** если горутина последняя в очереди ИЛИ ждала < 1ms — сбрасывает флаг `starving`. ^lock-starvation-exit
