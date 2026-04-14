- **GiST** (Generalized Search Tree) — сбалансированное дерево для типов с "близостью" и "пересечением": **геометрия, ranges, IP, PostGIS**
- **vs GIN**: GiST меньше размер, быстрее пишет, **медленнее ищет**. GIN наоборот
- Для full-text/массивов/JSONB — бери **GIN**. Для геоданных и диапазонов — **GiST**

---

**Использовать:**
- Геопространственные данные (PostGIS, координаты)
- Временные диапазоны (бронирования, расписания)
- IP ranges, CIDR
- Частые UPDATE (быстрее GIN)

**Не использовать:**
- Full-text search, массивы, JSONB (GIN лучше)
- Критична скорость SELECT (GIN быстрее)

```sql
CREATE INDEX idx_location ON places USING GiST(location);
CREATE INDEX idx_period ON bookings USING GiST(period);
```

## Связь
- [[GIN индекс]] — альтернатива: медленнее пишет, быстрее ищет
- [[Какие бывают индексы в PostgreSQL - сводка]] — decision tree по типам
