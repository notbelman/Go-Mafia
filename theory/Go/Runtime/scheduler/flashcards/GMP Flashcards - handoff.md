#flashcards/GMP/handoff

Почему блокирующий syscall — проблема для планировщика Go без handoff?
?
![[handoff#^handoff-problem]]

Опиши механизм handoff по шагам: что происходит с P, M и G при syscall.
?
![[handoff#^handoff-steps]]

После завершения syscall G1 ищет P. В каком порядке и что происходит если P не найден?
?
![[handoff#^handoff-after-syscall]]

Зачем P — отдельная сущность, а не просто поле в M? Ответ через handoff.
?
![[handoff#^handoff-compiler]]

Handoff происходит при каждом syscall? Какой порог у sysmon?
?
![[handoff#^handoff-sysmon-threshold]]

Почему short-lived syscalls могут не вызывать handoff?
?
![[handoff#^handoff-not-immediate]]

Почему M не может отдать syscall другому потоку? Объясни механизм.
?
![[handoff#^handoff-why-m-stays]]

Почему CGO не получает handoff? В чём отличие от обычного syscall?
?
![[handoff#^handoff-cgo]]

Что происходит с M после завершения syscall если нет свободного P?
?
![[handoff#^handoff-thread-pool]]

Кто вставляет код handoff и когда — компилятор или runtime? Когда именно?
?
![[handoff#^handoff-compiler]]

Что выведет этот код и почему?
```go
package main

import (
    "fmt"
    "os"
    "runtime"
    "time"
)

func main() {
    runtime.GOMAXPROCS(1)
    go func() {
        // блокирующий syscall
        os.ReadFile("/dev/null")
        fmt.Println("goroutine done")
    }()
    time.Sleep(50 * time.Millisecond)
    fmt.Println("main done")
    fmt.Println("threads:", runtime.NumCPU())
}
```
?
`goroutine done`, `main done`, `threads: <NumCPU>` — даже при `GOMAXPROCS(1)` (один P) горутина с syscall выполнится: P отдаётся другому M через handoff, main продолжает работу. `NumCPU()` возвращает физические ядра, не GOMAXPROCS.
![[handoff#^handoff-steps]]
