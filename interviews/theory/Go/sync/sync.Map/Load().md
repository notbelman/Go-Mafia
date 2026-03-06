```
1. Ищем в read (atomic, без лока)
   |
   +-- Нашли --> return value
   |
   +-- Не нашли, amended=false --> return nil, false
   |
   +-- Не нашли, amended=true:
       |
       2. Lock(mu)
       3. Ищем в dirty
       4. misses++
       5. Unlock(mu)
```
^load-algorithm

**Lock-free путь:** если ключ найден в read — никаких локов, никаких атомарных записей. Просто atomic.Load указателя + поиск в map. ^load-lockfree-path

**misses:** каждый промах (ключ не в read, пришлось идти в dirty) инкрементирует счётчик. Когда misses >= len(dirty) — срабатывает promotion. ^load-misses
