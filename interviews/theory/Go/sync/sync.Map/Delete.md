```
1. Ищем в read (atomic, без лока)
   |
   +-- Нашли --> entry.p = nil (atomic CAS)
   |             return
   |
   +-- Не нашли, amended=false --> return (ключа нет)
   |
   +-- Не нашли, amended=true:
       |
       2. Lock(mu)
       3. Ищем в dirty
       4. Если нашли --> entry.p = nil
       5. Unlock(mu)
```
^delete-algorithm

**Важно:** Delete НЕ удаляет entry из map. Только обнуляет указатель. ^delete-soft

Физическое удаление происходит при создании нового dirty — записи с nil помечаются expunged и не копируются. ^delete-physical
