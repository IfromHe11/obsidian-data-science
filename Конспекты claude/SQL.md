#sql #базы-данных #backend #собеседования #аналитика

# 🗄️ SQL — конспект для собеседований (RU)

> Полный конспект по SQL: от базового синтаксиса до оконных функций, индексов, транзакций и оптимизации запросов. Составлен с прицелом на технические собеседования (Data Analyst, Backend, Data Engineer).

## Содержание

- [[#Что такое SQL и типы команд]]
- [[#SELECT — базовый синтаксис]]
- [[#Агрегатные функции и GROUP BY HAVING]]
- [[#JOIN — соединения таблиц]]
- [[#Подзапросы Subqueries]]
- [[#Оконные функции Window Functions]]
- [[#CTE — Common Table Expressions]]
- [[#UNION vs UNION ALL]]
- [[#Ключи PRIMARY UNIQUE FOREIGN]]
- [[#Индексы]]
- [[#Нормализация баз данных]]
- [[#Транзакции и ACID]]
- [[#Уровни изоляции транзакций]]
- [[#NULL и работа с NULL]]
- [[#Представления VIEW и материализованные представления]]
- [[#Хранимые процедуры функции триггеры]]
- [[#Порядок выполнения SQL-запроса]]
- [[#Оптимизация запросов и EXPLAIN]]
- [[#Частые вопросы на собеседовании]]

---

## Что такое SQL и типы команд

> [!definition] Определение **SQL** (Structured Query Language) — декларативный язык запросов для управления данными в реляционных базах данных. Декларативность означает, что мы описываем **что** хотим получить, а не **как** это сделать (это забота СУБД).

SQL-команды делятся на подгруппы:

|Группа|Расшифровка|Команды|Назначение|
|---|---|---|---|
|**DDL**|Data Definition Language|`CREATE`, `ALTER`, `DROP`, `TRUNCATE`|Определение структуры БД|
|**DML**|Data Manipulation Language|`SELECT`, `INSERT`, `UPDATE`, `DELETE`|Работа с данными|
|**DCL**|Data Control Language|`GRANT`, `REVOKE`|Управление правами доступа|
|**TCL**|Transaction Control Language|`COMMIT`, `ROLLBACK`, `SAVEPOINT`|Управление транзакциями|

> [!example] Пример
> 
> ```sql
> -- DDL
> CREATE TABLE employees (
>     id SERIAL PRIMARY KEY,
>     name VARCHAR(100) NOT NULL,
>     department_id INT,
>     salary NUMERIC(10,2)
> );
> 
> -- DML
> INSERT INTO employees (name, department_id, salary)
> VALUES ('Иван Петров', 1, 85000);
> ```

> [!warning] Частая ошибка Путают `DELETE`, `TRUNCATE` и `DROP`:
> 
> - `DELETE` — удаляет строки построчно, можно откатить (rollback), можно с `WHERE`, медленнее.
> - `TRUNCATE` — мгновенно очищает таблицу, сбрасывает автоинкремент, в большинстве СУБД не откатывается внутри транзакции (зависит от СУБД), `WHERE` использовать нельзя.
> - `DROP` — удаляет саму таблицу вместе со структурой.

---

## SELECT — базовый синтаксис

> [!formula] Синтаксис
> 
> ```sql
> SELECT [DISTINCT] столбцы
> FROM таблица
> [JOIN ...]
> [WHERE условие]
> [GROUP BY столбцы]
> [HAVING условие_на_группы]
> [ORDER BY столбцы [ASC|DESC]]
> [LIMIT n [OFFSET m]];
> ```

> [!example] Пример
> 
> ```sql
> SELECT department_id, AVG(salary) AS avg_salary
> FROM employees
> WHERE salary > 30000
> GROUP BY department_id
> HAVING AVG(salary) > 50000
> ORDER BY avg_salary DESC
> LIMIT 5;
> ```

Ключевые операторы `WHERE`:

- `=`, `<>` (`!=`), `<`, `>`, `<=`, `>=`
- `BETWEEN x AND y`
- `IN (список)`
- `LIKE '%шаблон%'` (текстовый поиск), `ILIKE` — регистронезависимо (PostgreSQL)
- `IS NULL` / `IS NOT NULL`
- `AND`, `OR`, `NOT`

---

## Агрегатные функции и GROUP BY / HAVING

> [!definition] Определение **Агрегатные функции** сворачивают множество строк в одно значение: `COUNT()`, `SUM()`, `AVG()`, `MIN()`, `MAX()`. `GROUP BY` группирует строки по значению столбца(ов) перед применением агрегатов. `HAVING` фильтрует **уже сгруппированные** данные (в отличие от `WHERE`, который фильтрует строки до группировки).

> [!example] Пример
> 
> ```sql
> SELECT department_id,
>        COUNT(*) AS cnt,
>        SUM(salary) AS total_salary
> FROM employees
> GROUP BY department_id
> HAVING COUNT(*) > 3;
> ```

> [!warning] Частая ошибка В `SELECT` со `GROUP BY` нельзя выбирать столбцы, которые не входят ни в `GROUP BY`, ни в агрегатную функцию — иначе неясно, какое значение показывать для группы (в MySQL это иногда "проходит" молча и выдаёт случайное значение, в PostgreSQL — ошибка).

---

## JOIN — соединения таблиц

> [!definition] Определение `JOIN` объединяет строки из двух и более таблиц на основе связующего условия.

|Тип JOIN|Что возвращает|
|---|---|
|`INNER JOIN`|Только совпадающие строки в обеих таблицах|
|`LEFT JOIN`|Все строки левой таблицы + совпадения справа (или NULL)|
|`RIGHT JOIN`|Все строки правой таблицы + совпадения слева (или NULL)|
|`FULL JOIN`|Все строки обеих таблиц, несовпавшие поля — NULL|
|`CROSS JOIN`|Декартово произведение (каждая строка с каждой)|
|`SELF JOIN`|Таблица соединяется сама с собой|

> [!example] Пример
> 
> ```sql
> -- Сотрудники и их отделы (даже если отдел не указан)
> SELECT e.name, d.department_name
> FROM employees e
> LEFT JOIN departments d ON e.department_id = d.id;
> 
> -- SELF JOIN: найти пары сотрудников с одинаковой зарплатой
> SELECT a.name, b.name, a.salary
> FROM employees a
> JOIN employees b ON a.salary = b.salary AND a.id < b.id;
> ```

> [!important] Ключевой вывод Разница между `WHERE` и `ON` при `LEFT JOIN`: условие в `ON` применяется **до** присоединения (не отфильтровывает уже прицепленные NULL-строки), а условие в `WHERE` — **после** JOIN, и может неожиданно превратить `LEFT JOIN` в фактический `INNER JOIN`, отбросив строки с NULL.

---

## Подзапросы (Subqueries)

> [!definition] Определение **Подзапрос** — запрос внутри другого запроса. Бывают:
> 
> - **Скалярные** — возвращают одно значение
> - **Строчные/табличные** — возвращают набор строк
> - **Коррелированные (correlated)** — ссылаются на внешний запрос и выполняются для каждой строки внешнего запроса
> - **Некоррелированные** — выполняются один раз, независимо от внешнего запроса

> [!example] Пример
> 
> ```sql
> -- Некоррелированный подзапрос
> SELECT name, salary
> FROM employees
> WHERE salary > (SELECT AVG(salary) FROM employees);
> 
> -- Коррелированный подзапрос: сотрудники с зарплатой выше средней по своему отделу
> SELECT e.name, e.salary
> FROM employees e
> WHERE e.salary > (
>     SELECT AVG(e2.salary)
>     FROM employees e2
>     WHERE e2.department_id = e.department_id
> );
> ```

> [!warning] Частая ошибка Коррелированные подзапросы могут выполняться на каждую строку внешнего запроса → сильно проседает производительность на больших таблицах. Часто их можно и нужно заменить на `JOIN` или оконную функцию.

---

## Оконные функции (Window Functions)

> [!definition] Определение **Оконная функция** выполняет вычисление над набором строк ("окном"), связанных с текущей строкой, **не сворачивая** результат в одну строку (в отличие от `GROUP BY`).

> [!formula] Синтаксис
> 
> ```sql
> функция(...) OVER (
>     [PARTITION BY столбцы]
>     [ORDER BY столбцы]
>     [ROWS/RANGE BETWEEN ... AND ...]
> )
> ```

Популярные оконные функции:

- `ROW_NUMBER()` — уникальный порядковый номер строки в разбиении
- `RANK()` — ранг с "пропусками" при одинаковых значениях (1,2,2,4)
- `DENSE_RANK()` — ранг без пропусков (1,2,2,3)
- `LAG(col, n)` / `LEAD(col, n)` — значение из предыдущей/следующей строки
- `NTILE(n)` — разбивка на n групп (квартили, децили)
- `SUM()/AVG()/COUNT() OVER (...)` — накопительные (running) агрегаты

> [!example] Пример
> 
> ```sql
> SELECT
>     name,
>     department_id,
>     salary,
>     RANK() OVER (PARTITION BY department_id ORDER BY salary DESC) AS rank_in_dept,
>     AVG(salary) OVER (PARTITION BY department_id) AS dept_avg,
>     LAG(salary) OVER (PARTITION BY department_id ORDER BY salary) AS prev_salary
> FROM employees;
> ```

> [!important] Ключевой вывод Частый вопрос на собеседовании: «найти 2-ю (n-ю) максимальную зарплату» — классическая задача решается через `DENSE_RANK()`:
> 
> ```sql
> SELECT name, salary FROM (
>     SELECT name, salary,
>            DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
>     FROM employees
> ) t WHERE rnk = 2;
> ```

---

## CTE — Common Table Expressions

> [!definition] Определение **CTE** (`WITH ... AS (...)`) — именованный временный результирующий набор, существующий только в рамках одного запроса. Улучшает читаемость, позволяет строить рекурсивные запросы.

> [!example] Пример
> 
> ```sql
> WITH dept_avg AS (
>     SELECT department_id, AVG(salary) AS avg_salary
>     FROM employees
>     GROUP BY department_id
> )
> SELECT e.name, e.salary, d.avg_salary
> FROM employees e
> JOIN dept_avg d ON e.department_id = d.department_id
> WHERE e.salary > d.avg_salary;
> 
> -- Рекурсивный CTE: построение иерархии сотрудников
> WITH RECURSIVE org_chart AS (
>     SELECT id, name, manager_id, 1 AS level
>     FROM employees WHERE manager_id IS NULL
>     UNION ALL
>     SELECT e.id, e.name, e.manager_id, oc.level + 1
>     FROM employees e
>     JOIN org_chart oc ON e.manager_id = oc.id
> )
> SELECT * FROM org_chart;
> ```

---

## UNION vs UNION ALL

> [!important] Ключевой вывод
> 
> - `UNION` — объединяет результаты двух запросов и **убирает дубликаты** (требует сортировки/дедупликации → медленнее).
> - `UNION ALL` — объединяет **без** удаления дубликатов (быстрее). Использовать по умолчанию, если точно знаете, что дублей нет или они не важны. Оба требуют одинакового числа столбцов и совместимых типов данных в обоих запросах.

> [!example] Пример
> 
> ```sql
> SELECT name FROM employees_2023
> UNION ALL
> SELECT name FROM employees_2024;
> ```

---

## Ключи (PRIMARY, UNIQUE, FOREIGN)

|Ключ|Назначение|
|---|---|
|`PRIMARY KEY`|Уникально идентифицирует строку, не может быть NULL, в таблице только один|
|`UNIQUE`|Гарантирует уникальность значений, может быть NULL (обычно одно NULL допускается)|
|`FOREIGN KEY`|Ссылается на PRIMARY/UNIQUE KEY другой таблицы, обеспечивает ссылочную целостность|
|`COMPOSITE KEY`|Составной ключ из нескольких столбцов|

> [!example] Пример
> 
> ```sql
> CREATE TABLE orders (
>     id SERIAL PRIMARY KEY,
>     customer_id INT NOT NULL,
>     FOREIGN KEY (customer_id) REFERENCES customers(id)
>         ON DELETE CASCADE
> );
> ```

> [!warning] Частая ошибка Забывают указать `ON DELETE`/`ON UPDATE` поведение для внешних ключей (`CASCADE`, `SET NULL`, `RESTRICT`, `NO ACTION`) — из-за этого при удалении родительской записи может возникнуть ошибка целостности или «осиротевшие» строки.

---

## Индексы

> [!definition] Определение **Индекс** — структура данных (чаще всего B-Tree), ускоряющая поиск строк по значению столбца — ценой дополнительного места на диске и замедления `INSERT`/`UPDATE`/`DELETE` (индекс тоже нужно обновлять).

Основные типы:

- **B-Tree** — универсальный, для равенства и диапазонов (`=`, `<`, `BETWEEN`, `ORDER BY`)
- **Hash** — только для точного равенства (`=`)
- **GIN/GiST** (PostgreSQL) — для полнотекстового поиска, JSON, массивов
- **Составной (composite) индекс** — по нескольким столбцам, важен порядок столбцов
- **Уникальный индекс** — дополнительно гарантирует уникальность

> [!example] Пример
> 
> ```sql
> CREATE INDEX idx_employees_department ON employees(department_id);
> CREATE UNIQUE INDEX idx_employees_email ON employees(email);
> CREATE INDEX idx_orders_customer_date ON orders(customer_id, order_date);
> ```

> [!important] Ключевой вывод Правило "leftmost prefix" для составных индексов: индекс `(customer_id, order_date)` эффективно используется для запросов по `customer_id` и по `(customer_id, order_date)`, но **не** для запросов только по `order_date`.

> [!warning] Частая ошибка Индексируют все подряд столбцы «на всякий случай» — это замедляет запись данных и раздувает БД. Индекс имеет смысл, если столбец часто используется в `WHERE`, `JOIN`, `ORDER BY` и обладает высокой селективностью (много разных значений).

---

## Нормализация баз данных

> [!definition] Определение **Нормализация** — процесс организации таблиц для устранения избыточности и аномалий (вставки, обновления, удаления) через разбиение на связанные таблицы.

|Форма|Требование|
|---|---|
|**1NF**|Атомарность значений (нет списков/массивов в одной ячейке), уникальный первичный ключ|
|**2NF**|1NF + нет частичной зависимости неключевых атрибутов от части составного ключа|
|**3NF**|2NF + нет транзитивной зависимости (неключевой атрибут не зависит от другого неключевого)|
|**BCNF**|Усиленная 3NF: каждый детерминант — потенциальный ключ|

> [!example] Пример Ненормализованная таблица `orders(order_id, customer_name, customer_email, product_name, product_price)` хранит имя и email клиента в каждой строке заказа — избыточность. После нормализации: отдельные таблицы `customers`, `products`, `orders` со связями через FK.

> [!important] Ключевой вывод Денормализация иногда делается **намеренно** в аналитических/read-heavy системах (data warehouse, OLAP) ради скорости чтения — trade-off между скоростью записи/целостностью и скоростью чтения.

---

## Транзакции и ACID

> [!definition] Определение **Транзакция** — последовательность операций, выполняемая как единое неделимое целое: либо выполняется полностью, либо не выполняется вовсе.

> [!formula] ACID
> 
> - **Atomicity (атомарность)** — всё или ничего
> - **Consistency (согласованность)** — БД переходит из одного валидного состояния в другое
> - **Isolation (изолированность)** — параллельные транзакции не влияют друг на друга (степень регулируется уровнем изоляции)
> - **Durability (устойчивость)** — после commit изменения сохраняются даже при сбое

> [!example] Пример
> 
> ```sql
> BEGIN;
> UPDATE accounts SET balance = balance - 100 WHERE id = 1;
> UPDATE accounts SET balance = balance + 100 WHERE id = 2;
> COMMIT; -- или ROLLBACK при ошибке
> ```

---

## Уровни изоляции транзакций

|Уровень|Защищает от|
|---|---|
|**Read Uncommitted**|Ничего — возможны "грязные чтения" (dirty read)|
|**Read Committed**|Dirty read (стандарт по умолчанию в PostgreSQL/Oracle)|
|**Repeatable Read**|Dirty read + неповторяющееся чтение (non-repeatable read)|
|**Serializable**|Всё вышеперечисленное + фантомное чтение (phantom read)|

> [!definition] Определение аномалий
> 
> - **Dirty read** — чтение незакоммиченных изменений другой транзакции
> - **Non-repeatable read** — повторное чтение той же строки в рамках транзакции даёт другой результат
> - **Phantom read** — повторный запрос диапазона возвращает новые/исчезнувшие строки

> [!warning] Частая ошибка Путают Repeatable Read и Serializable: Repeatable Read защищает от изменения **уже прочитанных** строк, но не от появления **новых** строк, попадающих под условие (фантомы). Только Serializable гарантирует полную изоляцию (ценой производительности).

---

## NULL и работа с NULL

> [!important] Ключевой вывод `NULL` — это отсутствие значения, а не 0 или пустая строка. Любое сравнение с `NULL` через `=` или `<>` даёт `NULL` (не `TRUE`/`FALSE`), поэтому `WHERE column = NULL` **никогда** не сработает — нужно `IS NULL`.

> [!example] Пример
> 
> ```sql
> SELECT COALESCE(phone, 'нет телефона') AS phone FROM customers;
> SELECT NULLIF(discount, 0) FROM orders; -- вернёт NULL, если discount = 0
> SELECT * FROM employees WHERE manager_id IS NULL;
> ```

> [!warning] Частая ошибка `COUNT(*)` считает все строки, а `COUNT(column)` — только строки, где `column IS NOT NULL`. Это часто путает при подсчёте "заполненности" данных.

---

## Представления (VIEW) и материализованные представления

> [!definition] Определение **VIEW** — сохранённый SQL-запрос, который выглядит как виртуальная таблица; данные не хранятся, а вычисляются при каждом обращении. **Материализованное представление (Materialized View)** физически хранит результат и требует обновления (`REFRESH`).

> [!example] Пример
> 
> ```sql
> CREATE VIEW high_earners AS
> SELECT name, department_id, salary
> FROM employees
> WHERE salary > 100000;
> 
> CREATE MATERIALIZED VIEW dept_stats AS
> SELECT department_id, AVG(salary) AS avg_salary
> FROM employees GROUP BY department_id;
> 
> REFRESH MATERIALIZED VIEW dept_stats;
> ```

---

## Хранимые процедуры, функции, триггеры

> [!definition] Определение
> 
> - **Функция** — возвращает значение, может использоваться внутри `SELECT`
> - **Хранимая процедура** — может не возвращать значения, выполняет набор операций, вызывается через `CALL`
> - **Триггер** — код, автоматически выполняющийся при событии (`INSERT`/`UPDATE`/`DELETE`) на таблице

> [!example] Пример
> 
> ```sql
> CREATE OR REPLACE FUNCTION get_avg_salary(dept_id INT)
> RETURNS NUMERIC AS $$
>     SELECT AVG(salary) FROM employees WHERE department_id = dept_id;
> $$ LANGUAGE SQL;
> 
> CREATE TRIGGER trg_update_timestamp
> BEFORE UPDATE ON employees
> FOR EACH ROW
> EXECUTE FUNCTION set_updated_at();
> ```

---

## Порядок выполнения SQL-запроса

> [!formula] Логический порядок выполнения
> 
> ```
> 1. FROM / JOIN
> 2. WHERE
> 3. GROUP BY
> 4. HAVING
> 5. SELECT
> 6. DISTINCT
> 7. ORDER BY
> 8. LIMIT / OFFSET
> ```

> [!important] Ключевой вывод Именно поэтому в `WHERE` нельзя использовать алиас, объявленный в `SELECT` (SELECT выполняется позже), а в `ORDER BY` — можно (он выполняется после SELECT). Также поэтому `HAVING` фильтрует группы, а не строки, — он идёт после `GROUP BY`.

---

## Оптимизация запросов и EXPLAIN

> [!definition] Определение `EXPLAIN` (и `EXPLAIN ANALYZE` в PostgreSQL) показывает **план выполнения** запроса: какие индексы используются, порядок соединения таблиц, оценку количества строк, стоимость операций.

> [!example] Пример
> 
> ```sql
> EXPLAIN ANALYZE
> SELECT * FROM orders WHERE customer_id = 42;
> ```

Практические советы по оптимизации:

- Избегать `SELECT *` — выбирать только нужные столбцы
- Добавлять индексы на столбцы в `JOIN`, `WHERE`, `ORDER BY`
- Избегать функций над индексируемым столбцом в `WHERE` (`WHERE YEAR(date) = 2024` ломает индекс — лучше `WHERE date >= '2024-01-01' AND date < '2025-01-01'`)
- Использовать `EXISTS` вместо `IN` для больших подзапросов (часто эффективнее)
- Не использовать `LIKE '%текст%'` с ведущим `%` на больших таблицах — обычные индексы не помогают

---

## Частые вопросы на собеседовании

> [!important] Шпаргалка вопросов
> 
> 1. Чем отличается `WHERE` от `HAVING`?
> 2. В чём разница между `INNER` и `LEFT JOIN`?
> 3. Что такое индекс и когда он замедляет работу?
> 4. Объясните ACID своими словами.
> 5. Что такое оконная функция и чем она отличается от `GROUP BY`?
> 6. Как найти дубликаты в таблице? (`GROUP BY ... HAVING COUNT(*) > 1`)
> 7. Как найти N-ю по величине зарплату? (`DENSE_RANK`/`OFFSET`/подзапрос с `LIMIT`)
> 8. Чем `UNION` отличается от `UNION ALL`?
> 9. Что такое нормализация и зачем она нужна?
> 10. Что произойдёт при `LEFT JOIN` с условием во `WHERE` вместо `ON`?

---

## Связанные заметки

- [[Индексы]]
- [[Нормализация баз данных]]
- [[Транзакции и ACID]]
- [[Оконные функции Window Functions]]
- [[Оптимизация запросов и EXPLAIN]]
- [[NoSQL vs SQL]]
- [[A-B тесты и аналитика]]
- [[Python для анализа данных]]

---

---

# 🗄️ SQL — Interview Study Notes (EN)

#sql #databases #backend #interviews #analytics

> A complete SQL study note: from basic syntax to window functions, indexes, transactions, and query optimization. Focused on technical interview topics (Data Analyst, Backend, Data Engineer).

## Table of Contents

- [[#What is SQL and Command Types]]
- [[#SELECT — Basic Syntax]]
- [[#Aggregate Functions and GROUP BY HAVING]]
- [[#JOIN — Table Joins]]
- [[#Subqueries]]
- [[#Window Functions]]
- [[#CTE — Common Table Expressions (EN)]]
- [[#UNION vs UNION ALL (EN)]]
- [[#Keys PRIMARY UNIQUE FOREIGN]]
- [[#Indexes]]
- [[#Database Normalization]]
- [[#Transactions and ACID]]
- [[#Transaction Isolation Levels]]
- [[#NULL Handling]]
- [[#VIEW and Materialized Views]]
- [[#Stored Procedures Functions Triggers]]
- [[#Logical Query Execution Order]]
- [[#Query Optimization and EXPLAIN]]
- [[#Common Interview Questions]]

---

## What is SQL and Command Types

> [!definition] Definition **SQL** (Structured Query Language) is a declarative language for managing data in relational databases. Declarative means you describe **what** you want, not **how** to get it (that's the DBMS's job).

SQL commands are grouped into:

|Group|Meaning|Commands|Purpose|
|---|---|---|---|
|**DDL**|Data Definition Language|`CREATE`, `ALTER`, `DROP`, `TRUNCATE`|Defines DB structure|
|**DML**|Data Manipulation Language|`SELECT`, `INSERT`, `UPDATE`, `DELETE`|Works with data|
|**DCL**|Data Control Language|`GRANT`, `REVOKE`|Manages access rights|
|**TCL**|Transaction Control Language|`COMMIT`, `ROLLBACK`, `SAVEPOINT`|Manages transactions|

> [!example] Example
> 
> ```sql
> -- DDL
> CREATE TABLE employees (
>     id SERIAL PRIMARY KEY,
>     name VARCHAR(100) NOT NULL,
>     department_id INT,
>     salary NUMERIC(10,2)
> );
> 
> -- DML
> INSERT INTO employees (name, department_id, salary)
> VALUES ('John Smith', 1, 85000);
> ```

> [!warning] Common Mistake Confusing `DELETE`, `TRUNCATE`, and `DROP`:
> 
> - `DELETE` removes rows one by one, is transactional/rollback-able, supports `WHERE`, is slower.
> - `TRUNCATE` instantly empties a table, resets auto-increment, is typically not rollback-able within a transaction (DB-dependent), and doesn't support `WHERE`.
> - `DROP` removes the table itself, including its structure.

---

## SELECT — Basic Syntax

> [!formula] Syntax
> 
> ```sql
> SELECT [DISTINCT] columns
> FROM table
> [JOIN ...]
> [WHERE condition]
> [GROUP BY columns]
> [HAVING group_condition]
> [ORDER BY columns [ASC|DESC]]
> [LIMIT n [OFFSET m]];
> ```

> [!example] Example
> 
> ```sql
> SELECT department_id, AVG(salary) AS avg_salary
> FROM employees
> WHERE salary > 30000
> GROUP BY department_id
> HAVING AVG(salary) > 50000
> ORDER BY avg_salary DESC
> LIMIT 5;
> ```

Key `WHERE` operators:

- `=`, `<>` (`!=`), `<`, `>`, `<=`, `>=`
- `BETWEEN x AND y`
- `IN (list)`
- `LIKE '%pattern%'`, `ILIKE` — case-insensitive (PostgreSQL)
- `IS NULL` / `IS NOT NULL`
- `AND`, `OR`, `NOT`

---

## Aggregate Functions and GROUP BY / HAVING

> [!definition] Definition **Aggregate functions** collapse multiple rows into one value: `COUNT()`, `SUM()`, `AVG()`, `MIN()`, `MAX()`. `GROUP BY` groups rows by column value(s) before aggregation is applied. `HAVING` filters **already grouped** data (unlike `WHERE`, which filters rows before grouping).

> [!example] Example
> 
> ```sql
> SELECT department_id,
>        COUNT(*) AS cnt,
>        SUM(salary) AS total_salary
> FROM employees
> GROUP BY department_id
> HAVING COUNT(*) > 3;
> ```

> [!warning] Common Mistake With `GROUP BY`, `SELECT` cannot include columns that aren't in `GROUP BY` or wrapped in an aggregate — otherwise it's unclear which value to show per group (MySQL sometimes allows this silently and returns an arbitrary value; PostgreSQL throws an error).

---

## JOIN — Table Joins

> [!definition] Definition `JOIN` combines rows from two or more tables based on a related condition.

|Join Type|Returns|
|---|---|
|`INNER JOIN`|Only matching rows in both tables|
|`LEFT JOIN`|All rows from the left table + matches on the right (or NULL)|
|`RIGHT JOIN`|All rows from the right table + matches on the left (or NULL)|
|`FULL JOIN`|All rows from both tables; unmatched fields are NULL|
|`CROSS JOIN`|Cartesian product (every row with every row)|
|`SELF JOIN`|A table joined with itself|

> [!example] Example
> 
> ```sql
> -- Employees and their departments (even if department is missing)
> SELECT e.name, d.department_name
> FROM employees e
> LEFT JOIN departments d ON e.department_id = d.id;
> 
> -- SELF JOIN: find pairs of employees with equal salary
> SELECT a.name, b.name, a.salary
> FROM employees a
> JOIN employees b ON a.salary = b.salary AND a.id < b.id;
> ```

> [!important] Key Takeaway The difference between `WHERE` and `ON` with `LEFT JOIN`: the `ON` condition applies **before** the join (it does not filter out already-attached NULL rows), while a `WHERE` condition applies **after** the join and can unexpectedly turn a `LEFT JOIN` into an effective `INNER JOIN` by dropping NULL rows.

---

## Subqueries

> [!definition] Definition A **subquery** is a query nested inside another query. Types:
> 
> - **Scalar** — returns a single value
> - **Row/table** — returns a set of rows
> - **Correlated** — references the outer query and runs once per outer row
> - **Non-correlated** — runs once, independent of the outer query

> [!example] Example
> 
> ```sql
> -- Non-correlated subquery
> SELECT name, salary
> FROM employees
> WHERE salary > (SELECT AVG(salary) FROM employees);
> 
> -- Correlated subquery: employees earning above their dept's average
> SELECT e.name, e.salary
> FROM employees e
> WHERE e.salary > (
>     SELECT AVG(e2.salary)
>     FROM employees e2
>     WHERE e2.department_id = e.department_id
> );
> ```

> [!warning] Common Mistake Correlated subqueries can run once per outer row → performance drops significantly on large tables. They can often (and should) be rewritten as a `JOIN` or a window function.

---

## Window Functions

> [!definition] Definition A **window function** performs a calculation across a set of rows ("window") related to the current row, **without collapsing** the result into one row (unlike `GROUP BY`).

> [!formula] Syntax
> 
> ```sql
> function(...) OVER (
>     [PARTITION BY columns]
>     [ORDER BY columns]
>     [ROWS/RANGE BETWEEN ... AND ...]
> )
> ```

Common window functions:

- `ROW_NUMBER()` — unique sequential number within a partition
- `RANK()` — rank with gaps for ties (1,2,2,4)
- `DENSE_RANK()` — rank without gaps (1,2,2,3)
- `LAG(col, n)` / `LEAD(col, n)` — value from the previous/next row
- `NTILE(n)` — splits rows into n buckets (quartiles, deciles)
- `SUM()/AVG()/COUNT() OVER (...)` — running/cumulative aggregates

> [!example] Example
> 
> ```sql
> SELECT
>     name,
>     department_id,
>     salary,
>     RANK() OVER (PARTITION BY department_id ORDER BY salary DESC) AS rank_in_dept,
>     AVG(salary) OVER (PARTITION BY department_id) AS dept_avg,
>     LAG(salary) OVER (PARTITION BY department_id ORDER BY salary) AS prev_salary
> FROM employees;
> ```

> [!important] Key Takeaway A classic interview task — "find the 2nd (Nth) highest salary" — is solved cleanly with `DENSE_RANK()`:
> 
> ```sql
> SELECT name, salary FROM (
>     SELECT name, salary,
>            DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
>     FROM employees
> ) t WHERE rnk = 2;
> ```

---

## CTE — Common Table Expressions (EN)

> [!definition] Definition A **CTE** (`WITH ... AS (...)`) is a named temporary result set that exists only within a single query. Improves readability and enables recursive queries.

> [!example] Example
> 
> ```sql
> WITH dept_avg AS (
>     SELECT department_id, AVG(salary) AS avg_salary
>     FROM employees
>     GROUP BY department_id
> )
> SELECT e.name, e.salary, d.avg_salary
> FROM employees e
> JOIN dept_avg d ON e.department_id = d.department_id
> WHERE e.salary > d.avg_salary;
> 
> -- Recursive CTE: building an org chart
> WITH RECURSIVE org_chart AS (
>     SELECT id, name, manager_id, 1 AS level
>     FROM employees WHERE manager_id IS NULL
>     UNION ALL
>     SELECT e.id, e.name, e.manager_id, oc.level + 1
>     FROM employees e
>     JOIN org_chart oc ON e.manager_id = oc.id
> )
> SELECT * FROM org_chart;
> ```

---

## UNION vs UNION ALL (EN)

> [!important] Key Takeaway
> 
> - `UNION` combines results from two queries and **removes duplicates** (requires sorting/dedup → slower).
> - `UNION ALL` combines results **without** removing duplicates (faster). Prefer it by default when you know there are no duplicates or they don't matter. Both require the same number of columns and compatible data types in each query.

> [!example] Example
> 
> ```sql
> SELECT name FROM employees_2023
> UNION ALL
> SELECT name FROM employees_2024;
> ```

---

## Keys (PRIMARY, UNIQUE, FOREIGN)

|Key|Purpose|
|---|---|
|`PRIMARY KEY`|Uniquely identifies a row, cannot be NULL, only one per table|
|`UNIQUE`|Guarantees value uniqueness, can be NULL (usually one NULL allowed)|
|`FOREIGN KEY`|References a PRIMARY/UNIQUE key in another table, enforces referential integrity|
|`COMPOSITE KEY`|A key made up of multiple columns|

> [!example] Example
> 
> ```sql
> CREATE TABLE orders (
>     id SERIAL PRIMARY KEY,
>     customer_id INT NOT NULL,
>     FOREIGN KEY (customer_id) REFERENCES customers(id)
>         ON DELETE CASCADE
> );
> ```

> [!warning] Common Mistake Forgetting to specify `ON DELETE`/`ON UPDATE` behavior for foreign keys (`CASCADE`, `SET NULL`, `RESTRICT`, `NO ACTION`) — this can cause integrity errors or orphaned rows when a parent record is deleted.

---

## Indexes

> [!definition] Definition An **index** is a data structure (usually a B-Tree) that speeds up row lookups by column value — at the cost of extra disk space and slower `INSERT`/`UPDATE`/`DELETE` (the index must be updated too).

Main types:

- **B-Tree** — general-purpose, for equality and ranges (`=`, `<`, `BETWEEN`, `ORDER BY`)
- **Hash** — exact equality only (`=`)
- **GIN/GiST** (PostgreSQL) — for full-text search, JSON, arrays
- **Composite index** — spans multiple columns; column order matters
- **Unique index** — additionally enforces uniqueness

> [!example] Example
> 
> ```sql
> CREATE INDEX idx_employees_department ON employees(department_id);
> CREATE UNIQUE INDEX idx_employees_email ON employees(email);
> CREATE INDEX idx_orders_customer_date ON orders(customer_id, order_date);
> ```

> [!important] Key Takeaway The "leftmost prefix" rule for composite indexes: an index on `(customer_id, order_date)` is effectively used for queries filtering by `customer_id` or by `(customer_id, order_date)`, but **not** for queries filtering only by `order_date`.

> [!warning] Common Mistake Indexing every column "just in case" — this slows down writes and bloats the database. Indexes make sense when a column is frequently used in `WHERE`, `JOIN`, `ORDER BY` and has high selectivity (many distinct values).

---

## Database Normalization

> [!definition] Definition **Normalization** is the process of organizing tables to eliminate redundancy and anomalies (insert, update, delete) by splitting data into related tables.

|Form|Requirement|
|---|---|
|**1NF**|Atomic values (no lists/arrays in a single cell), unique primary key|
|**2NF**|1NF + no partial dependency of non-key attributes on part of a composite key|
|**3NF**|2NF + no transitive dependency (a non-key attribute doesn't depend on another non-key attribute)|
|**BCNF**|Stronger 3NF: every determinant is a candidate key|

> [!example] Example A denormalized table `orders(order_id, customer_name, customer_email, product_name, product_price)` stores customer name/email in every order row — redundant. After normalization: separate `customers`, `products`, `orders` tables linked via FK.

> [!important] Key Takeaway Denormalization is sometimes done **intentionally** in analytical/read-heavy systems (data warehouses, OLAP) for read speed — a trade-off between write speed/integrity and read speed.

---

## Transactions and ACID

> [!definition] Definition A **transaction** is a sequence of operations executed as a single indivisible unit: either all of it succeeds, or none of it does.

> [!formula] ACID
> 
> - **Atomicity** — all or nothing
> - **Consistency** — the DB moves from one valid state to another
> - **Isolation** — concurrent transactions don't interfere with each other (degree controlled by isolation level)
> - **Durability** — once committed, changes survive even a crash

> [!example] Example
> 
> ```sql
> BEGIN;
> UPDATE accounts SET balance = balance - 100 WHERE id = 1;
> UPDATE accounts SET balance = balance + 100 WHERE id = 2;
> COMMIT; -- or ROLLBACK on error
> ```

---

## Transaction Isolation Levels

|Level|Protects Against|
|---|---|
|**Read Uncommitted**|Nothing — dirty reads possible|
|**Read Committed**|Dirty reads (default in PostgreSQL/Oracle)|
|**Repeatable Read**|Dirty reads + non-repeatable reads|
|**Serializable**|All of the above + phantom reads|

> [!definition] Anomaly Definitions
> 
> - **Dirty read** — reading uncommitted changes from another transaction
> - **Non-repeatable read** — re-reading the same row within a transaction returns a different result
> - **Phantom read** — re-running a range query returns new/vanished rows

> [!warning] Common Mistake Confusing Repeatable Read with Serializable: Repeatable Read protects **already-read** rows from changing, but not from **new** rows appearing that match the query condition (phantoms). Only Serializable guarantees full isolation (at a performance cost).

---

## NULL Handling

> [!important] Key Takeaway `NULL` means "absence of a value," not 0 or an empty string. Any comparison with `NULL` using `=` or `<>` evaluates to `NULL` (not `TRUE`/`FALSE`), so `WHERE column = NULL` **never** matches — use `IS NULL` instead.

> [!example] Example
> 
> ```sql
> SELECT COALESCE(phone, 'no phone') AS phone FROM customers;
> SELECT NULLIF(discount, 0) FROM orders; -- returns NULL if discount = 0
> SELECT * FROM employees WHERE manager_id IS NULL;
> ```

> [!warning] Common Mistake `COUNT(*)` counts all rows, while `COUNT(column)` counts only rows where `column IS NOT NULL`. This often trips people up when checking data completeness.

---

## VIEW and Materialized Views

> [!definition] Definition A **VIEW** is a saved SQL query that behaves like a virtual table; data isn't stored, it's computed on each access. A **Materialized View** physically stores the result and needs to be refreshed (`REFRESH`).

> [!example] Example
> 
> ```sql
> CREATE VIEW high_earners AS
> SELECT name, department_id, salary
> FROM employees
> WHERE salary > 100000;
> 
> CREATE MATERIALIZED VIEW dept_stats AS
> SELECT department_id, AVG(salary) AS avg_salary
> FROM employees GROUP BY department_id;
> 
> REFRESH MATERIALIZED VIEW dept_stats;
> ```

---

## Stored Procedures, Functions, Triggers

> [!definition] Definition
> 
> - **Function** — returns a value, can be used inside a `SELECT`
> - **Stored procedure** — may not return a value, performs a set of operations, called via `CALL`
> - **Trigger** — code automatically executed on an event (`INSERT`/`UPDATE`/`DELETE`) on a table

> [!example] Example
> 
> ```sql
> CREATE OR REPLACE FUNCTION get_avg_salary(dept_id INT)
> RETURNS NUMERIC AS $$
>     SELECT AVG(salary) FROM employees WHERE department_id = dept_id;
> $$ LANGUAGE SQL;
> 
> CREATE TRIGGER trg_update_timestamp
> BEFORE UPDATE ON employees
> FOR EACH ROW
> EXECUTE FUNCTION set_updated_at();
> ```

---

## Logical Query Execution Order

> [!formula] Logical Execution Order
> 
> ```
> 1. FROM / JOIN
> 2. WHERE
> 3. GROUP BY
> 4. HAVING
> 5. SELECT
> 6. DISTINCT
> 7. ORDER BY
> 8. LIMIT / OFFSET
> ```

> [!important] Key Takeaway This is exactly why you can't use a `SELECT`-defined alias in `WHERE` (SELECT runs later), but you can in `ORDER BY` (it runs after SELECT). It's also why `HAVING` filters groups, not rows — it runs after `GROUP BY`.

---

## Query Optimization and EXPLAIN

> [!definition] Definition `EXPLAIN` (and `EXPLAIN ANALYZE` in PostgreSQL) shows the query's **execution plan**: which indexes are used, join order, estimated row counts, and operation costs.

> [!example] Example
> 
> ```sql
> EXPLAIN ANALYZE
> SELECT * FROM orders WHERE customer_id = 42;
> ```

Practical optimization tips:

- Avoid `SELECT *` — select only the columns you need
- Add indexes on columns used in `JOIN`, `WHERE`, `ORDER BY`
- Avoid wrapping an indexed column in a function inside `WHERE` (`WHERE YEAR(date) = 2024` breaks index usage — prefer `WHERE date >= '2024-01-01' AND date < '2025-01-01'`)
- Use `EXISTS` instead of `IN` for large subqueries (often more efficient)
- Avoid `LIKE '%text%'` with a leading `%` on large tables — standard indexes can't help

---

## Common Interview Questions

> [!important] Cheat Sheet
> 
> 1. What's the difference between `WHERE` and `HAVING`?
> 2. What's the difference between `INNER` and `LEFT JOIN`?
> 3. What is an index, and when does it slow things down?
> 4. Explain ACID in your own words.
> 5. What is a window function, and how does it differ from `GROUP BY`?
> 6. How do you find duplicates in a table? (`GROUP BY ... HAVING COUNT(*) > 1`)
> 7. How do you find the Nth highest salary? (`DENSE_RANK`/`OFFSET`/subquery with `LIMIT`)
> 8. What's the difference between `UNION` and `UNION ALL`?
> 9. What is normalization and why is it needed?
> 10. What happens with a `LEFT JOIN` when the condition is in `WHERE` instead of `ON`?

---

## Related Notes

- [[Indexes]]
- [[Database Normalization]]
- [[Transactions and ACID]]
- [[Window Functions]]
- [[Query Optimization and EXPLAIN]]
- [[NoSQL vs SQL]]
- [[A/B Testing and Analytics]]
- [[Python for Data Analysis]]