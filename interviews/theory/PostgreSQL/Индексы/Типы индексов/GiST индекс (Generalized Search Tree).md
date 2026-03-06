
**Что это:** Сбалансированное дерево для типов с понятием "близости" и "пересечения" (геометрия, ranges, IP)

**Использовать:**

- Геопространственные данные (PostGIS, координаты)
- Временные диапазоны (бронирования, расписания)
- IP ranges, CIDR
- Частые UPDATE (быстрее GIN)

**Не использовать:**

- Full-text search, массивы, JSONB (GIN лучше)
- Критична скорость SELECT (GIN быстрее)

**vs GIN:** меньше размер, быстрее пишет, медленнее ищет

```sql
CREATE INDEX idx_location ON places USING GiST(location);
CREATE INDEX idx_period ON bookings USING GiST(period);
```