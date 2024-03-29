# Домашнее задание к занятию 2. «SQL» - Паромов Роман

## Задача 1

### Используя Docker, поднимите инстанс PostgreSQL (версию 12) c 2 volume, в который будут складываться данные БД и бэкапы.

Приведите получившуюся команду или docker-compose-манифест.
```
version: "3"
services:
  postgres:
    image: postgres:12
    container_name: postgres
    environment:             
      POSTGRES_DB: "test_db "
      POSTGRES_PASSWORD: "0000"
      PGDATA: /data/database
    restart: unless-stopped
    volumes:
      - db-data:/data/database
      - db-backup:/data/backup
    ports:
      - "5432:5432"
    networks:
      - subnet1

networks:
  subnet1:

volumes:
  db-data:
  db-backup:
```

## Задача 2

### В БД из задачи 1:

* создайте пользователя test-admin-user и БД test_db;
* в БД test_db создайте таблицу orders и clients (спeцификация таблиц ниже);
* предоставьте привилегии на все операции пользователю test-admin-user на таблицы БД test_db;
* создайте пользователя test-simple-user;
* предоставьте пользователю test-simple-user права на SELECT/INSERT/UPDATE/DELETE этих таблиц БД test_db.

Таблица orders:

* id (serial primary key);
* наименование (string);
* цена (integer).

Таблица clients:

* id (serial primary key);
* фамилия (string);
* страна проживания (string, index);
* заказ (foreign key orders).

Приведите:

* итоговый список БД после выполнения пунктов выше; описание таблиц (describe);
![](https://github.com/Romera14/11-01_BD/blob/main/bases%20and%20tables.png)

* SQL-запрос для выдачи списка пользователей с правами над таблицами test_db;
![](https://github.com/Romera14/11-01_BD/blob/main/sql_prvlg_users.png)

* список пользователей с правами над таблицами test_db.
![](https://github.com/Romera14/11-01_BD/blob/main/dp_prvlg_users.png)

## Задача 3

### Используя SQL-синтаксис, наполните таблицы следующими тестовыми данными:

Таблица orders

Наименование	цена

Шоколад	10

Принтер	3000

Книга	500

Монитор	7000

Гитара	4000

Таблица clients

ФИО	Страна проживания

Иванов Иван Иванович	USA

Петров Петр Петрович	Canada

Иоганн Себастьян Бах	Japan

Ронни Джеймс Дио	Russia

Ritchie Blackmore	Russia

Используя SQL-синтаксис:

вычислите количество записей для каждой таблицы.
Приведите в ответе:

- запросы,
- результаты их выполнения.
![](https://github.com/Romera14/11-01_BD/blob/main/push%20tables.png)

## Задача 4

### Часть пользователей из таблицы clients решили оформить заказы из таблицы orders.

Используя foreign keys, свяжите записи из таблиц, согласно таблице:


ФИО	Заказ

#### Иванов Иван Иванович	Книга

Петров Петр Петрович	Монитор

Иоганн Себастьян Бах	Гитара

### Приведите SQL-запросы для выполнения этих операций.

Приведите SQL-запрос для выдачи всех пользователей, которые совершили заказ, а также вывод этого запроса.

Подсказка: используйте директиву UPDATE.
![](https://github.com/Romera14/11-01_BD/blob/main/user%20paid.png)

## Задача 5

### Получите полную информацию по выполнению запроса выдачи всех пользователей из задачи 4 (используя директиву EXPLAIN).

Приведите получившийся результат и объясните, что значат полученные значения.
![](https://github.com/Romera14/11-01_BD/blob/main/explain.png)
* Seq Scan - сканирование таблицы в параллельном режиме.
* cost - оценка стоимости (отношение потраченных ресурсов ко времени).
* rows - количество записей использованных для получения выходных данных
* среднее количество байт в возвращенной строке.
* filter - отфильтрованные строки.

## Задача 6

### Создайте бэкап БД test_db и поместите его в volume, предназначенный для бэкапов (см. задачу 1).

Остановите контейнер с PostgreSQL, но не удаляйте volumes.

Поднимите новый пустой контейнер с PostgreSQL.

Восстановите БД test_db в новом контейнере.

Приведите список операций, который вы применяли для бэкапа данных и восстановления.
* ```pg_dump -h 0.0.0.0 -U postgres test_db2 > data/backup/test_db2.dump```
* ```docker-compose down --volumes``` пришлось удалить вместе volumes, потому что как запустить паралельно еще один плейбук я не нашел.
* ```docker-compose up -d```
* ```psql -h 0.0.0.0 -U postgres```
* ```create database test_db2```
* ```psql -h 0.0.0.0 -U postgres test_db2 < data/backup/test_db2.dump```

Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

