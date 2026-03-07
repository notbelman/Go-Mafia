**Case с nil-каналом игнорируется** — это главная фича. ^nil-select-main-feature

| Ситуация                         | Результат                        |
| :------------------------------- | :------------------------------- |
| Есть nil и не-nil каналы         | Только не-nil участвуют в выборе |
| Все каналы `nil`, есть `default` | Выполняется `default`            |
| Все каналы `nil`, нет `default`  | **Deadlock**                     | ^nil-select-table

Когда все каналы в `select` равны `nil` и нет `default` — рантайм детектирует deadlock и паникует. ^nil-select-all-nil-deadlock
