```
Store(key, value)
       │
       ▼
┌─────────────────────────┐
│ Ключ есть в read?       │
└─────────────────────────┘
       │
       ├── Да ──► trySwap (atomic)
       │              │
       │              ├── entry.p != expunged -─► CAS(old, new) ──► return
       │              │
       │              └── entry.p == expunged ──► fail, идём под лок
       │
       └── Нет ──► идём под лок
                      │
                      ▼
              ┌───────────────┐
              │   Lock(mu)    │
              └───────────────┘
                      │
                      ▼
              ┌─────────────────────────┐
              │ Ключ есть в read?       │
              └─────────────────────────┘
                      │
                      ├── Да, expunged ──► unexpunge + добавить в dirty + записать
                      │
                      ├── Да, не expunged ──► просто записать
                      │
                      └── Нет в read:
                              │
                              ▼
                      ┌─────────────────────────┐
                      │ Ключ есть в dirty?      │
                      └─────────────────────────┘
                              │
                              ├── Да ──► записать в dirty
                              │
                              └── Нет (новый ключ):
                                      │
                                      ├── dirty == nil? ──► dirtyLocked() (создать dirty из read)
                                      │
                                      └── dirty[key] = newEntry(value)
                                          amended = true
                      │
                      ▼
              ┌───────────────┐
              │  Unlock(mu)   │
              └───────────────┘
```
^store-algorithm

**Почему CAS в trySwap:** ^store-cas-reason

Проверяет что entry.p != expunged. Если за время между проверкой и записью кто-то сделал delete + создал новый dirty — entry стал expunged. CAS это ловит и фейлит → идём под лок. ^store-cas-detail

**Double-check после лока:** после взятия лока снова проверяем read — состояние могло измениться пока мы ждали лок. ^store-double-check

**Когда Store lock-free:** только если ключ уже есть в read И entry не expunged. Тогда — CAS без mutex. ^store-lockfree-condition

**Новый ключ всегда под локом:** если ключа нет ни в read, ни в dirty — берём mutex, при необходимости вызываем dirtyLocked(), пишем в dirty, amended = true. ^store-new-key
