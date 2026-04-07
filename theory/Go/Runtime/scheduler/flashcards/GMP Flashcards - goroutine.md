#flashcards/GMP/goroutine

Какой начальный размер стека горутины и почему это важно по сравнению с потоком ОС?
?
![[G (Goroutine)#^g-stack-initial]]
<!--SR:!2026-02-24,1,230-->

Каков максимальный размер стека горутины на 64-bit системе?
?
![[G (Goroutine)#^g-stack-max]]
<!--SR:!2026-02-24,1,230-->

Поток ОС получает X стека сразу, горутина начинает с Y. Назови конкретные числа.
?
![[G (Goroutine)#^g-stack-os-thread]]
<!--SR:!2026-02-24,1,230-->

Что происходит с регистрами при переключении контекста горутины?
?
![[G (Goroutine)#^g-switch-registers]]
+
![[G (Goroutine)#^g-switch-sp-bp]]
<!--SR:!2026-02-24,1,230-->

Что переключается при контекст-свитче горутины — stack pointer, base pointer, или копируется стек?
?
![[G (Goroutine)#^g-switch-registers]]
+
![[G (Goroutine)#^g-switch-sp-bp]]
<!--SR:!2026-02-24,1,218-->

Почему при переключении горутин стек не копируется?
?
![[G (Goroutine)#^g-stack-no-copy]]
+
![[G (Goroutine)#^g-switch-registers]]
+
![[G (Goroutine)#^g-switch-sp-bp]]
<!--SR:!2026-02-24,1,230-->

Опиши полный жизненный цикл горутины от `go func()` до переиспользования.
?
![[G (Goroutine)#^g-lifecycle]]
<!--SR:!2026-02-24,1,218-->

Почему завершённые горутины не уничтожаются сразу? Что с ними происходит?
?
![[G (Goroutine)#^g-gfree-pool]]
<!--SR:!2026-02-24,1,230-->

Почему goid не является публичным API? Какой антипаттерн это предотвращает?
?
![[G (Goroutine)#^g-goid-private]]
<!--SR:!2026-02-24,1,218-->

Для чего используются поля `waitsince` и `waitreason` в структуре runtime.g?
?
![[G (Goroutine)#^g-wait-pprof]]
<!--SR:!2026-02-24,1,230-->

Что такое поле `preempt` в структуре G и когда оно проверяется?
?
![[G (Goroutine)#^g-preempt-flag]]
<!--SR:!2026-02-24,1,230-->

Какие поля есть в структуре runtime.g? Назови основные с назначением.
?
![[G (Goroutine)#^g-struct-fields]]
<!--SR:!2026-02-24,1,230-->

Что выведет этот код?
```go
package main

import (
    "fmt"
    "runtime"
    "sync"
)

func main() {
    var wg sync.WaitGroup
    results := make([]int, 5)
    for i := 0; i < 5; i++ {
        wg.Add(1)
        i := i
        go func() {
            defer wg.Done()
            results[i] = i * 2
        }()
    }
    wg.Wait()
    fmt.Println(results)
    fmt.Println(runtime.NumGoroutine())
}
```
?
`[0 2 4 6 8]` и `1` — каждая горутина захватывает свою копию `i` благодаря `i := i`. После `wg.Wait()` все горутины завершились, остаётся только main, поэтому `NumGoroutine() = 1`.
![[G (Goroutine)#^g-lifecycle]]
<!--SR:!2026-02-24,1,230-->
