**Что это:** функция которая создаёт новый dirty когда его нет. ^dirtylocked-def

**Когда вызывается:** После promotion dirty = nil. Кто-то делает Store нового ключа — надо создать dirty. ^dirtylocked-when

**Что делает:** ^dirtylocked-steps

1. Создаёт пустую map
2. Копирует всё из read
3. Записи с nil ПРОПУСКАЕТ (и помечает их expunged)

**Пример:**
```
read:  { a: *val, b: nil, c: *val }  ← b удалён, но ещё в read
dirty: nil

        ↓ Store("d") — нужен dirty ↓

read:  { a: *val, b: expunged, c: *val }  ← nil стал expunged
dirty: { a: *val, c: *val, d: *val }       ← b не скопировали
```
^dirtylocked-example

**Зачем expunged:** чтобы знать что ключ есть в read, но его НЕТ в dirty. Это позволяет Store на такой ключ сначала unexpunge, добавить в dirty, потом записать. ^dirtylocked-expunged-role
