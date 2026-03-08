- **95%** случаев — B-tree. JSONB/массивы/full-text — **GIN**. Геометрия/ranges — **GiST**. Огромные логи — **BRIN**. Hash — почти никогда
- Если сомневаешься — **используй B-tree**
- Синтаксис: `CREATE INDEX idx ON table USING GIN(column)`. Без USING — B-tree по дефолту

---

## 5 основных типов

| Тип | Когда | Операции | Размер |
|---|---|---|---|
| **B-tree** | Дефолт для всего | `=`, `<`, `>`, `BETWEEN`, `ORDER BY`, `LIKE 'prefix%'` | ~30-40% |
| **Hash** | Только `=` на длинных строках | `=` | ~20-30% |
| **GIN** | JSONB, массивы, full-text | `@>`, `?`, `@@`, `&&` | ~50-200% |
| **GiST** | Геометрия, ranges, IP | Пересечения, близость | ~30-100% |
| **BRIN** | Огромные append-only таблицы | `>`, `<`, `BETWEEN` | ~0.1% |

## Decision tree

```
Что индексируем?
│
├─ Обычные колонки (int, text, timestamp)?
│  ├─ >100M строк + естественная сортировка → BRIN
│  └─ Обычная таблица → B-tree (дефолт)
│
├─ JSONB поле? → GIN
├─ Массив (tags)? → GIN
├─ Full-text search? → GIN (на tsvector)
├─ Геометрия? → GiST
└─ Ranges (время, IP)? → GiST
```

## Создание

```sql
CREATE INDEX idx ON table(column);                    -- B-tree
CREATE INDEX idx ON table USING GIN(column);          -- GIN
CREATE INDEX idx ON table USING GiST(column);         -- GiST
CREATE INDEX idx ON table USING BRIN(column);          -- BRIN
CREATE INDEX idx ON table USING HASH(column);          -- Hash
```

## Связь
- [[theory/PostgreSQL + SQL/indexes/B-tree индекс]] — подробности дефолтного типа
- [[GIN индекс]] — инвертированный индекс
- [[GiST индекс]] — геоданные, ranges
- [[BRIN индекс]] — append-only таблицы
- [[theory/PostgreSQL + SQL/indexes/Hash индекс]] — только =, почти никогда
