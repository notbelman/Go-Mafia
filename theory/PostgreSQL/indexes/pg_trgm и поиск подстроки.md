- **pg_trgm** — расширение для поиска подстроки: `LIKE '%text%'`, `ILIKE`, similarity. B-tree для этого **бесполезен**
- Работает через **триграммы**: разбивает строку на 3-символьные куски. `'hello'` → `{'  h', ' he', 'hel', 'ell', 'llo', 'lo '}`
- Индекс: **GIN** (быстрее поиск, медленнее запись) или **GiST** (быстрее запись, медленнее поиск)

---

## Установка

```sql
CREATE EXTENSION pg_trgm;
```

## Создание индекса

```sql
-- GIN — для частых SELECT, редких INSERT (рекомендуется)
CREATE INDEX idx_name_trgm ON users USING GIN(name gin_trgm_ops);

-- GiST — для частых INSERT, или если нужен ORDER BY similarity
CREATE INDEX idx_name_trgm ON users USING GiST(name gist_trgm_ops);
```

## Какие запросы ускоряет

```sql
-- Поиск подстроки
SELECT * FROM users WHERE name LIKE '%алекс%';         -- ✅
SELECT * FROM users WHERE name ILIKE '%АЛЕКС%';         -- ✅

-- Similarity (похожесть)
SELECT * FROM users WHERE similarity(name, 'алексей') > 0.3;  -- ✅

-- Обычный LIKE с префиксом — тут B-tree справится и без pg_trgm
SELECT * FROM users WHERE name LIKE 'алекс%';          -- B-tree ОК
```

## Как работает

```
'hello' → триграммы: {'  h', ' he', 'hel', 'ell', 'llo', 'lo '}
'hell'  → триграммы: {'  h', ' he', 'hel', 'ell', 'll '}

Общие триграммы: {'  h', ' he', 'hel', 'ell'} → similarity = 4/7 ≈ 0.57

LIKE '%ell%':
  'ell' — триграмма → ищем в GIN индексе → находим строки содержащие 'ell'
```

## Порог similarity

```sql
-- Дефолтный порог: 0.3
SELECT show_limit();

-- Изменить для сессии
SET pg_trgm.similarity_threshold = 0.5;

-- Оператор %: name % 'алексей' = similarity > threshold
SELECT * FROM users WHERE name % 'алексей';
```

## Размер индекса

GIN + pg_trgm = **большой** индекс (каждая строка → N триграмм → N записей в индексе). На таблице 1M строк со средним name 10 символов: индекс может быть **2-5x** от размера колонки.

## Связь
- [[GIN индекс]] — pg_trgm использует GIN для хранения триграмм
- [[Expression индексы (functional)]] — можно комбинировать: `GIN(LOWER(name) gin_trgm_ops)`
- [[Селективность и издержки индексов]] — pg_trgm индекс дорогой по месту
