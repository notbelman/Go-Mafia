#flashcards/sync-map/entry_states

Какие три состояния может принимать entry.p? Что каждое означает?
?
![[entry_состояния#^entry-states-table]]

entry.p = nil. Удалён ли ключ? В каких структурах он находится?
?
![[entry_состояния#^entry-states-table]]

entry.p = expunged. Удалён ли ключ? В каких структурах он находится?
?
![[entry_состояния#^entry-states-table]]

В чём разница между nil и expunged? Почему недостаточно только nil?
?
![[entry_состояния#^entry-expunged-purpose]]

Опиши все переходы между состояниями entry: Active → Deleted → Expunged → revival.
?
![[entry_состояния#^entry-transitions]]

Когда происходит переход nil → expunged?
?
![[entry_состояния#^entry-expunged-detail]]

Что такое revival (воскрешение) entry? Какие шаги происходят?
?
![[entry_состояния#^entry-revival]]
