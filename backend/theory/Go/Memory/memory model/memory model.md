## Memory Model в Go

#### [[Reordering инструкций]]
CPU и компилятор переупорядочивают инструкции, видимость изменений

#### [[Барьеры памяти]]
Memory barrier/fence — запрет переупорядочивания, acquire/release семантика

#### [[MESI vs барьеры]]
Протокол когерентности кэша MESI, как барьеры вписываются

#### [[Happens-before через atomic]]
atomic операции создают happens-before, sync.Mutex тоже
