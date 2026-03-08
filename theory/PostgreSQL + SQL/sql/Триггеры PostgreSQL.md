- **Триггер** — функция, автоматически вызываемая при INSERT/UPDATE/DELETE. **BEFORE** (можно изменить/отменить строку) или **AFTER** (строка уже записана, для side effects)
- **FOR EACH ROW** (на каждую строку) vs **FOR EACH STATEMENT** (один раз на весь запрос). Row-level имеют доступ к `NEW` и `OLD`
- Подводные камни: **скрытая логика** (дебаг ада), **каскадные триггеры** (триггер вызывает триггер), **замедление DML** (каждый INSERT/UPDATE/DELETE запускает код)
- Правило: **избегай бизнес-логики** в триггерах. Используй для аудита, автозаполнения полей, поддержания инвариантов

---

## Синтаксис

```sql
-- 1. Создать функцию
CREATE OR REPLACE FUNCTION update_modified() RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = NOW();  -- NEW = новая версия строки
  RETURN NEW;               -- RETURN NEW обязателен для BEFORE
END;
$$ LANGUAGE plpgsql;

-- 2. Привязать к таблице
CREATE TRIGGER trg_update_modified
  BEFORE UPDATE ON users        -- BEFORE: можно модифицировать NEW
  FOR EACH ROW                  -- на каждую строку
  EXECUTE FUNCTION update_modified();
```

## BEFORE vs AFTER

| | BEFORE | AFTER |
|:--|:--|:--|
| Когда | **до** записи в heap | **после** записи |
| Доступ к NEW/OLD | да, можно **изменить** NEW | да, но изменение **не повлияет** |
| RETURN NULL | **отменяет** операцию (строка не запишется) | не имеет смысла |
| Use case | валидация, автозаполнение, модификация | аудит, уведомления, обновление связанных таблиц |

```sql
-- BEFORE: автозаполнение slug
CREATE FUNCTION auto_slug() RETURNS TRIGGER AS $$
BEGIN
  NEW.slug = LOWER(REPLACE(NEW.title, ' ', '-'));
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- AFTER: аудит
CREATE FUNCTION audit_log() RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO audit (table_name, action, old_data, new_data, changed_at)
  VALUES (TG_TABLE_NAME, TG_OP, row_to_json(OLD), row_to_json(NEW), NOW());
  RETURN NULL;  -- для AFTER возвращаемое значение игнорируется
END;
$$ LANGUAGE plpgsql;
```

## FOR EACH ROW vs FOR EACH STATEMENT

```sql
-- UPDATE users SET status = 'active' WHERE age > 18;  (затронуло 500 строк)

FOR EACH ROW:       -- функция вызовется 500 раз, NEW/OLD доступны
FOR EACH STATEMENT: -- функция вызовется 1 раз, NEW/OLD НЕдоступны
```

Statement-level: для логирования "был UPDATE на таблице users", без деталей по строкам.

## Специальные переменные

| Переменная | Что |
|:--|:--|
| `NEW` | новая версия строки (INSERT, UPDATE) |
| `OLD` | старая версия строки (UPDATE, DELETE) |
| `TG_OP` | `'INSERT'`, `'UPDATE'`, `'DELETE'`, `'TRUNCATE'` |
| `TG_TABLE_NAME` | имя таблицы |
| `TG_WHEN` | `'BEFORE'`, `'AFTER'`, `'INSTEAD OF'` |

## Условный триггер (WHEN)

```sql
CREATE TRIGGER trg_notify_vip
  AFTER UPDATE ON orders
  FOR EACH ROW
  WHEN (NEW.total > 10000 AND OLD.total <= 10000)  -- только при переходе порога
  EXECUTE FUNCTION notify_vip_order();
```

## INSTEAD OF (для VIEW)

```sql
-- VIEW не поддерживает прямой INSERT → триггер перенаправляет
CREATE TRIGGER trg_insert_view
  INSTEAD OF INSERT ON user_orders_view
  FOR EACH ROW
  EXECUTE FUNCTION insert_into_tables();
```

## Подводные камни

- **Скрытая логика**: `INSERT INTO users (...)` — откуда знать что запустится триггер? Дебаг-ад
- **Каскад**: триггер на таблице A делает INSERT в B → триггер на B делает UPDATE A → ...
- **Производительность**: каждый DML = вызов PL/pgSQL. На bulk INSERT 1M строк — 1M вызовов
- **Транзакция**: триггер выполняется **внутри** транзакции вызвавшего запроса. Ошибка в триггере → ROLLBACK всей транзакции

## Когда использовать

**Да**: updated_at, аудит-лог, поддержание денормализованного поля (counter cache), soft delete (INSTEAD OF DELETE → UPDATE set deleted_at).

**Нет**: бизнес-логика (валидация заказа, расчёт скидок) — это должно быть в коде приложения.

## Связь
- [[Блокировки PostgreSQL]] — триггеры работают внутри транзакции, могут влиять на блокировки
- [[MVCC в PostgreSQL]] — AFTER триггер видит уже записанную версию строки
- [[Миграции без даунтайма]] — триггеры полезны при backfill данных
