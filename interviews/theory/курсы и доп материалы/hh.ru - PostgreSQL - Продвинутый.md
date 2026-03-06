## Вопрос 1

**Нужно получить список пользователей и количество их подтверждённых заказов. Результат — отсортированный по убыванию количества заказов. Какой из запросов это реализует?**

✅ **Ответ:**

```sql
select user_id, count(order_id) from orders
where status = 'confirmed'
group by user_id order by count(order_id) desc
```
## Вопрос 2

**Выберите корректный вывод по следующему запросу:**

```sql
select date_trunc('quarter', created_at), ...
from revenue
group by 1
```

✅ **Ответ:** Группировка по кварталам
## Вопрос 3

**В таблице customers есть поле `interests` типа `text[]`. Нужно добавить `'sports'` только тем клиентам, у кого его нет. Коллега предложил:**

```sql
update customers
set interests = interests || 'sports'
where not 'sports' = any(interests);
```

**Какой из вариантов корректнее и безопаснее?**

✅ **Ответ:**

```sql
update customers set interests =
  array_append(interests, 'sports')
  where not 'sports' = any(interests)
```
## Вопрос 4

**Почему следующий запрос может вернуть больше строк, чем таблица employees?**

```sql
select *
from employees e
left join salaries s on e.id = s.emp_id
```

✅ **Ответ:** В таблице `salaries` могут быть дубликаты по `emp_id`
## Вопрос 5

**Что произойдёт при выполнении этого запроса?**

```sql
select *
from products
where id not in (select product_id from archive)
```

✅ **Ответ:** Вернёт товары, не попавшие в архив
## Вопрос 6

**В таблице `some_table` 1 млн записей. На `id` — B-Tree индекс, на `name` — GIN для полнотекстового поиска. Какой запрос оптимально использует индекс при выборке менее 0.1%?**

✅ **Ответ:**

```sql
select * from some_table where id = 1000;
```
## Вопрос 7

**Таблица events (10 млн строк), частые запросы:**

```sql
select * from events where category = 'system'
  and created_at > now() - interval '1 day'
```

**Какой индекс лучше?**

✅ **Ответ:**

```sql
create index on events (category, created_at)
```
## Вопрос 8

**Транзакция T1 при уровне изоляции Read Committed:**

```sql
BEGIN;
SELECT balance FROM accounts WHERE id = 42;
-- пауза, T2 делает UPDATE balance = 200 и COMMIT
SELECT balance FROM accounts WHERE id = 42;
COMMIT;
```

**Что увидит T1?**

✅ **Ответ:** Разные значения balance в двух `SELECT`
## Вопрос 9

**Что может быть проблемой при сравнении чисел типа float?**

```sql
select * from results where score = 0.1
```

✅ **Ответ:** 0.1 может быть представлен неточно в памяти
## Вопрос 10

**Что вернёт запрос?**

```sql
select department, count(*)
from employees
group by department
having count(*) > 1;
```

✅ **Ответ:** Только департаменты, где 2 и более сотрудников
## Вопрос 11

**При удалении пользователя `delete from users where id = 123` наблюдается задержка. В таблице orders: `foreign key (user_id) references users(id) on delete cascade`. Как оценить влияние каскадного удаления?**

✅ **Ответ:**

```sql
EXPLAIN ANALYZE delete from users where id = 123
```
## Вопрос 12

**В таблице orders 1 млн записей. На `customer_id` есть индекс B-Tree. Запрос с фильтрацией по customer_id выбирает 90% строк. Какой тип сканирования?**

✅ **Ответ:** seq scan
## Вопрос 13

**Таблица `action_log`. После релиза частота вставки записей с `action = 'logIn'` возросла в 10 раз. Нужно временно не вставлять такие записи, сохранив существующие. Как?**

✅ **Ответ:** Создать BEFORE INSERT trigger, запрещающий 'logIn'
## Вопрос 14

**`GRANT ALL PRIVILEGES ON kinds TO manuel;` — выполнена пользователем, который не является суперпользователем и не является владельцем таблицы. Результат?**

✅ **Ответ:** Команда завершится ошибкой, так как исполнитель команды не является владельцем таблицы
## Вопрос 15

**Вы хотите разделить таблицу users на два шарда по регионам при использовании PostgreSQL с Citus. Какой шаг необходим?**

✅ **Ответ:**

```sql
select create_distributed_table('users', 'region')
```
## Вопрос 16

**Вы настраиваете потоковую репликацию PostgreSQL. Какую строку конфигурации необходимо добавить в postgresql. conf на мастере?**

✅ **Ответ:** `wal_level = replica`
## Вопрос 17

**Что делает рекурсивный СТЕ в этом фрагменте кода?**
```sql
with recursive t(n) as (
select 1 union all
select n + 1 from t where n < 3
)
select * from t;
```

✅ **Ответ:** Возвращает число от 1 до 3
## Вопрос 18

**B PostgreSQL вы создаёте индекс в транзакции над таблицей users. Во время выполнения этой транзакции другая транзакция пытается выполнить одну из следующих команд над той же таблицей. Какая команда выполнится без конфликтов?**
```sql
-- Первая. транзакция
begin;
create index id_users_name on users (name);

-- втовая тоанзакция
«выбранная команда»;
```

✅ **Ответ:** 
```sql 
select * from users;
```
## Вопрос 19

**Почему при уровне Repeatable Read может произойти ошибка сериализации?**
```sql
BEGIN;
SELECT
FROM stock WHERE product_id = 1;
-- в другом сеансе UPDATE той же строки
SELECT * FROM stock WHERE product_id = 1;
COMMIT;
```

✅ **Ответ:** Потому что другой сеанс изменил строку, нарушив консистентность

## Вопрос 20

**Нужно посчитать зарплату сотрудников среди активных сотрудников. Какой запрос подойдет?**

✅ **Ответ:** 
```sql 
select department, sum(salary) from employees where active = true group by department
```

## Вопрос 21

**Имеется следующая структура представленная на изображении. Было решено во все существующие записи в таблице ... "чтобы не было дубликатов". Как это достичь?**

✅ **Ответ:** 
```sql 
update table role alter column permissions = array_append(permissions, 'read')
```

## Вопрос 22

**В чем потенциальная проблема этого запроса?**
```sql
select count(*)

from orders o

join orders_items i on o.id = i.order_id
```

✅ **Ответ:** `COUNT(*)` может посчитать дубликаты 

## Вопрос 23

**Почему запрос может не использовать индекс?**
```sql
select * from payments where status != 'confirmed'
```

✅ **Ответ:** `!=` приводит к полному сканированию таблицы