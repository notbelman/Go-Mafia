## GC в Go

#### [[Обзор GC]]
Tri-color mark-and-sweep, concurrent GC, фазы STW

#### [[Tri-color marking]]
White/grey/black объекты, алгоритм маркировки, invariant

#### [[Write barrier]]
Hybrid write barrier (Dijkstra + Yuasa), зачем при concurrent mark

#### [[Mark Assist]]
Горутина помогает GC при высоком allocation rate

#### [[GC Pacer]]
Регулятор триггера GC, heap goal, формула Go 1.18+

#### [[GOGC и GOMEMLIMIT]]
GOGC=100 — удвоение heap, GOMEMLIMIT soft limit (Go 1.19+)

#### [[Tracing (базовый STW)]]
STW подход: scan stack roots, mark, sweep vs copying

#### [[Finalizers]]
runtime.SetFinalizer, когда полезен, почему опасен, KeepAlive

#### [[Что такое мусор и когда можно не собирать]]
Reachability, когда GC можно не делать, ручной vs автоматический

#### [[Поколения (generational GC)]]
Generational hypothesis, почему Go не использует, стек ≈ young gen

#### [[Reference counting]]
RC как альтернатива GC, циклические ссылки, почему Go выбрал tracing

#### [[Lazy allocation и RSS vs VSS]]
Lazy alloc Linux, RSS vs VSS, ballast-хак

#### [[Green Tea GC (new)]]
Go 1.26 default: page-based marking, FIFO work list, seen+scanned bits, 10–40% CPU

#### [[Green Tea - Vector acceleration (new)]]
AVX-512 scanning kernel, VGF2P8AFFINEQB, +10% на Intel Ice Lake / AMD Zen 4+
