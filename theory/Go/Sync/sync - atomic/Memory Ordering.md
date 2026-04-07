**Проблема**: CPU и компилятор переставляют операции для оптимизации ^mem-reorder-problem

  ```
  // написал:          // CPU выполнил:
  data = "ready"       flag = true
  flag = true          data = "ready"
  ```

Для одного потока — без разницы, результат тот же. ^mem-reorder-single-thread

Для двух потоков — баги: ^mem-reorder-multithread

  ```
  // Поток 1:              // Поток 2:
  data = "ready"           if flag {
  flag = true                  print(data) // пусто!
                           }
  ```

  CPU переставил → Поток 2 видит flag=true, но data ещё пустой ^mem-reorder-bug

**Решение**: atomic операции — барьер, через который нельзя переставлять ^mem-barrier-solution

  ```
  // Поток 1:              // Поток 2:
  data = "ready"           if flag.Load() {      // барьер
  flag.Store(true) // ←        print(data)       // гарантия "ready"
                           }
  ```

  Store/Load — барьеры. Всё что ДО Store видно ПОСЛЕ Load. ^mem-store-load-guarantee

В Go: atomic = sequential consistency ^mem-go-seq-consistency

Все потоки видят операции в одном порядке. Думать не надо — просто работает. ^mem-go-simple

## Связь
- [[Go Memory Model и Memory Barriers]] — memory barriers
- [[Happens-Before в Go]] — гарантии порядка
- [[sync - atomic]] — atomic = barrier
