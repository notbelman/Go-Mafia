| entry.p    | Удалено? | Где ключ                                   |
| :--------- | :------- | :----------------------------------------- |
| `*value`   | Нет      | read И dirty ИЛИ только dirty (новый ключ) |
| `nil`      | Да       | read И dirty                               |
| `expunged` | Да       | ТОЛЬКО read                                |
^entry-states-table

**Переходы:**
```
Active --[Delete]--> Deleted --[создание dirty]--> Expunged
                         |                              |
                         +-------[Store того же]--------+
                                    (revival)
```
^entry-transitions

**Зачем expunged:** ^entry-expunged-purpose

При создании нового dirty копируем read, но удалённые записи (nil) не копируем — помечаем их expunged. Так они не попадут в dirty и исчезнут при следующем promotion. ^entry-expunged-detail

**Revival (воскрешение expunged записи):** если сделать Store на expunged ключ, entry сначала unexpunge (p = nil), затем добавляется в dirty, затем записывается значение. ^entry-revival
