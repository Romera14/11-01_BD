# Домашнее задание к занятию 4. «PostgreSQL» - Паромов Роман

## Задача 1

### Используя Docker, поднимите инстанс PostgreSQL (версию 13). Данные БД сохраните в volume.

Подключитесь к БД PostgreSQL, используя psql.

Воспользуйтесь командой \? для вывода подсказки по имеющимся в psql управляющим командам.

Найдите и приведите управляющие команды для:

* Вывода списка БД ```\l+```
* Подключения к БД ```\c```
* Вывода списка таблиц ```\dtS```
* Вывода описания содержимого таблиц ```\dS+```
* Выхода из psql ```\q```

## Задача 2

Используя psql, создайте БД test_database.

Изучите бэкап БД.

Восстановите бэкап БД в test_database. ```psql -U postgres -h 0.0.0.0 -f ./test_db.dump  test_database```

Перейдите в управляющую консоль psql внутри контейнера.

Подключитесь к восстановленной БД и проведите операцию ANALYZE для сбора статистики по таблице.
```
test_database=# ANALYZE VERBOSE public.orders;
INFO:  analyzing "public.orders"
INFO:  "orders": scanned 1 of 1 pages, containing 8 live rows and 0 dead rows; 8 rows in sample, 8 estimated total rows
ANALYZE
```

Используя таблицу pg_stats, найдите столбец таблицы orders с наибольшим средним значением размера элементов в байтах.

Приведите в ответе команду, которую вы использовали для вычисления, и полученный результат.
```
test_database=# ANALYZE VERBOSE public.orders;
INFO:  analyzing "public.orders"
INFO:  "orders": scanned 1 of 1 pages, containing 8 live rows and 0 dead rows; 8 rows in sample, 8 estimated total rows
ANALYZE
test_database=# SELECT avg_width FROM pg_stats WHERE tablename='orders';
 avg_width 
-----------
         4
        16
         4
(3 строки)
```

Задача 3

Архитектор и администратор БД выяснили, что ваша таблица orders разрослась до невиданных размеров и поиск по ней занимает долгое время. Вам как успешному выпускнику курсов DevOps в Нетологии предложили провести разбиение таблицы на 2: шардировать на orders_1 - price>499 и orders_2 - price<=499.

Предложите SQL-транзакцию для проведения этой операции.
```
test_database=# CREATE TABLE orders_1 (CHECK (price > 499)) INHERITS (orders);
CREATE TABLE
test_database=# INSERT INTO orders_1 SELECT * FROM orders WHERE price > 499;
INSERT 0 3
test_database=# CREATE TABLE orders_2 (CHECK (price <= 499)) INHERITS (orders);
CREATE TABLE
test_database=# INSERT INTO orders_2 SELECT * FROM orders WHERE price <= 499;
INSERT 0 5
test_database=# DELETE FROM ONLY orders;
DELETE 8
test_database=# \dt
            Список отношений
 Схема  |   Имя    |   Тип   | Владелец 
--------+----------+---------+----------
 public | orders   | таблица | postgres
 public | orders_1 | таблица | postgres
 public | orders_2 | таблица | postgres
(3 строки)
```

Можно ли было изначально исключить ручное разбиение при проектировании таблицы orders?
Можно если прописать RULE через if else.

Задача 4

Используя утилиту pg_dump, создайте бекап БД test_database.
```
pg_dump -h 0.0.0.0 -U postgres test_database > ./data/backup/test_database_backup.sql
```

Как бы вы доработали бэкап-файл, чтобы добавить уникальность значения столбца title для таблиц test_database?
* Можно добавить свойство ```unique```
