
**5 основных типов индексов:**

| Тип | Когда использовать | Операции | Размер |
|-----|-------------------|----------|--------|
| **B-tree** | Дефолт для всего | `=`, `<`, `>`, `BETWEEN`, `ORDER BY`, `LIKE 'prefix%'` | ~30-40% |
| **Hash** | Только `=` на больших строках | `=` | ~20-30% |
| **GIN** | JSONB, массивы, full-text | `@>`, `?`, `@@`, `&&` | ~50-200% |
| **GiST** | Геометрия, ranges, IP | Пересечения, близость, `&&`, `<<=` | ~30-100% |
| **BRIN** | Огромные таблицы с сортировкой | `>`, `<`, `BETWEEN` | ~0.1% |

**Decision tree:**
```
Что индексируем?
│
├─ Обычные колонки (int, text, timestamp)?
│  ├─ Большая таблица (>100M строк) + естественная сортировка?
│  │  └─ BRIN
│  └─ Обычная таблица?
│     └─ B-tree (дефолт)
│
├─ JSONB поле?
│  └─ GIN
│
├─ Массив (tags, categories)?
│  └─ GIN
│
├─ Full-text search?
│  └─ GIN (на tsvector)
│
├─ Геометрия (точки, полигоны)?
│  └─ GiST
│
└─ Ranges (временные диапазоны, IP)?
   └─ GiST
```

**Создание индекса (синтаксис):**
```sql
-- B-tree (по умолчанию)
CREATE INDEX idx_name ON table(column);

-- Явное указание типа
CREATE INDEX idx_name ON table USING GIN(column);
CREATE INDEX idx_name ON table USING GiST(column);
CREATE INDEX idx_name ON table USING BRIN(column);
CREATE INDEX idx_name ON table USING HASH(column);
```

**Золотое правило:**
- 95% случаев - B-tree
- JSONB/массивы/full-text - GIN
- Геометрия/ranges - GiST
- Огромные логи - BRIN
- Hash - почти никогда

⚠️ Если сомневаешься - используй B-tree!