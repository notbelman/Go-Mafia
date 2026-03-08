Расширение для автоматизации управления партициями.

## Что делает

- Автоматически создаёт новые партиции (daily/weekly/monthly)
- Автоматически удаляет старые по retention policy
- Создаёт default партицию
- Background worker - не нужен внешний cron

## Зачем нужно

Без pg_partman: забыл создать партицию на следующий месяц - INSERT падает с ошибкой. С pg_partman: партиции создаются автоматически заранее.

## Когда использовать

- Time-series данные с постоянным ростом
- Нужен retention policy
- Не хочется писать свой cron для создания партиций
## Пример
```sql
-- установка
CREATE SCHEMA partman;
CREATE EXTENSION pg_partman SCHEMA partman;

-- настройка партицирования (один раз)
SELECT partman.create_parent(
    'public.orders',      -- таблица
    'created_at',         -- ключ партиции
    'native',             -- declarative partitioning
    'daily'               -- интервал
);

-- retention - автоудаление старше 30 дней
UPDATE partman.part_config 
SET retention = '30 days',
    retention_keep_table = false  -- дропать, не просто detach
WHERE parent_table = 'public.orders';

-- запуск maintenance вручную (или через BGW автоматом)
CALL partman.run_maintenance_proc();
```