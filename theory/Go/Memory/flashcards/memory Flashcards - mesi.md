#flashcards/memory/mesi

Что гарантирует MESI? Чего он НЕ гарантирует?
?
![[MESI vs барьеры#^mesi-guarantee]]
![[MESI vs барьеры#^mesi-no-order-guarantee]]

Что гарантируют барьеры памяти в отличие от MESI?
?
![[MESI vs барьеры#^mesi-barriers-order]]
![[MESI vs барьеры#^mesi-barrier-guarantee]]

Опиши цепочку: проблема кэшей → MESI → новая проблема → store buffer → новая проблема → барьеры. Почему каждое решение порождает новую проблему?
?
![[MESI vs барьеры#^mesi-chain-pattern]]

Что произойдёт с ядром 1 в этом сценарии без барьера?
```
Ядро 0: X = 42, Y = 1
Ядро 1: if Y == 1 → читает X
```
?
![[MESI vs барьеры#^mesi-no-barrier-problem]]

Как нужно изменить код ядра 0, чтобы ядро 1 гарантированно видело X=42 когда видит Y=1?
?
![[MESI vs барьеры#^mesi-barrier-guarantee]]

Объясни аналогию с почтой: чем MESI похож на обычную почту, а барьеры — на гарантированный порядок доставки?
?
![[MESI vs барьеры#^mesi-analogy-mesi]]
![[MESI vs барьеры#^mesi-analogy-barrier]]
![[MESI vs барьеры#^mesi-analogy-no-barrier]]

Почему ядро ждёт подтверждения инвалидации в MESI и как store buffer решает эту проблему производительности?
?
![[MESI vs барьеры#^mesi-chain-pattern]]
