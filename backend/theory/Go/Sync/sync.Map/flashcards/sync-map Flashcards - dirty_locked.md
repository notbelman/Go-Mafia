#flashcards/sync-map/dirty_locked

Что такое dirtyLocked()? Когда вызывается?
?
![[dirtyLocked#^dirtylocked-def]]
![[dirtyLocked#^dirtylocked-when]]

Какие три шага выполняет dirtyLocked()?
?
![[dirtyLocked#^dirtylocked-steps]]

Что происходит с записями nil при вызове dirtyLocked()?
?
![[dirtyLocked#^dirtylocked-steps]]

После promotion dirty = nil. Кто-то делает Store нового ключа. Что произойдёт?
?
![[dirtyLocked#^dirtylocked-when]]

Покажи на примере: read = {a: *val, b: nil, c: *val}, dirty: nil. Вызвали dirtyLocked(). Что получится?
?
![[dirtyLocked#^dirtylocked-example]]

Зачем dirtyLocked() ставит expunged вместо того чтобы просто пропустить nil-записи?
?
![[dirtyLocked#^dirtylocked-expunged-role]]
