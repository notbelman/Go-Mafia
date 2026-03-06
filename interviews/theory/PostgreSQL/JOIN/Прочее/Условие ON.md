
Не только id = id! Любое булево выражение.
```sql
-- Пересечение диапазонов IP (PostgreSQL ip4r)
SELECT s.id, c.city
FROM users_stats AS s
JOIN cities_ip_ranges AS c
    ON c.ip_range && s.ip

-- BETWEEN
JOIN prices ON date BETWEEN start_date AND end_date

-- Сложные условия
JOIN t2 ON t1.x > t2.y AND t1.type = 'A'
```

**Трюк:**
```sql
JOIN t2 ON true
-- это то же самое что CROSS JOIN
```

⚠️ 99% разработчиков думают что ON только для id = id