## SQL техники

#### [[CTE (Common Table Expression)]]
WITH clause, рекурсивные CTE, материализация, когда inline vs материализовать

#### [[Оконные функции]]
OVER, PARTITION BY, ORDER BY, ROWS/RANGE, ROW_NUMBER, RANK, LAG/LEAD, NTILE

#### [[HAVING vs WHERE]]
WHERE фильтрует строки до агрегации, HAVING — после, производительность

#### [[Триггеры PostgreSQL]]
BEFORE/AFTER, FOR EACH ROW/STATEMENT, когда использовать, подводные камни

#### [[Миграции без даунтайма]]
Expand-migrate-contract паттерн, NOT VALID, concurrent index creation

#### [[Write amplification]]
Проблема лишних записей, HOT updates, индексы и UPDATE

#### [[SELECT all антипаттерн]]
SELECT * — почему плохо, производительность, covering index
