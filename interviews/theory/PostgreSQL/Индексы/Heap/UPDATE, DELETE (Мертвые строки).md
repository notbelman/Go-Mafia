**Dead tuples (bloat)**  PostgreSQL не изменяет и не удаляет данные физически (из-за MVCC).

**DELETE:**
- Строка (tuple) помечается как мёртвая
- В индексе указатель на неё остаётся

**UPDATE:**
- Старая версия строки помечается как мёртвая
- Создаётся новая строка с новым TID
- В индекс добавляется новый указатель

```
До UPDATE:
  Индекс: alice@gmail.com → (0,2)
  Heap:   (0,2) = {id:3, email:'alice@gmail.com'}

После UPDATE email → 'alice_new@gmail.com':
  Индекс: alice@gmail.com     → (0,2)  ← мёртвый указатель
          alice_new@gmail.com → (0,4)  ← новый
  Heap:   (0,2) = мёртвая строка
          (0,4) = {id:3, email:'alice_new@gmail.com'}
```
**Итог:** после UPDATE/DELETE в heap и индексе накапливается мусор → нужен VACUUM.