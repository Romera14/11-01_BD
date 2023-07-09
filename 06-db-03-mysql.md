# Домашнее задание к занятию 3. «MySQL» - Паромов Роман

## Задача 1

### Используя Docker, поднимите инстанс MySQL (версию 8). Данные БД сохраните в volume.

Изучите бэкап БД и восстановитесь из него.

Перейдите в управляющую консоль mysql внутри контейнера.

Используя команду \h, получите список управляющих команд.

Найдите команду для выдачи статуса БД и приведите в ответе из её вывода версию сервера БД.
```
mysql> \s
--------------
mysql  Ver 8.0.33 for Linux on x86_64 (MySQL Community Server - GPL)
```

Подключитесь к восстановленной БД и получите список таблиц из этой БД.

Приведите в ответе количество записей с price > 300.
```
mysql> select * from orders where price > 300;
+----+----------------+-------+
| id | title          | price |
+----+----------------+-------+
|  2 | My little pony |   500 |
+----+----------------+-------+
1 row in set (0.01 sec)
```

В следующих заданиях мы будем продолжать работу с этим контейнером.

## Задача 2

### Создайте пользователя test в БД c паролем test-pass, используя:

* плагин авторизации mysql_native_password
* срок истечения пароля — 180 дней
* количество попыток авторизации — 3
* максимальное количество запросов в час — 100
* аттрибуты пользователя:
* Фамилия "Pretty"
* Имя "James".
* Предоставьте привелегии пользователю test на операции SELECT базы test_db.
```
mysql> CREATE USER 'test'@'localhost'
    ->     IDENTIFIED WITH mysql_native_password BY 'test-pass'
    ->     WITH MAX_QUERIES_PER_HOUR 100
    ->     FAILED_LOGIN_ATTEMPTS 3 
    ->     PASSWORD EXPIRE INTERVAL 180 DAY
    ->     ATTRIBUTE '{"name":"James", "lname":"Pretty"}';
Query OK, 0 rows affected (0.04 sec)

mysql> GRANT SELECT ON test_db.* TO test@localhost;
Query OK, 0 rows affected, 1 warning (0.01 sec)
```

Используя таблицу INFORMATION_SCHEMA.USER_ATTRIBUTES, получите данные по пользователю test и приведите в ответе к задаче.
```
mysql> SELECT * FROM INFORMATION_SCHEMA.USER_ATTRIBUTES WHERE USER = 'test';
+------+-----------+--------------------------------------+
| USER | HOST      | ATTRIBUTE                            |
+------+-----------+--------------------------------------+
| test | localhost | {"name": "James", "lname": "Pretty"} |
+------+-----------+--------------------------------------+
1 row in set (0.01 sec)
```

## Задача 3

Установите профилирование SET profiling = 1. Изучите вывод профилирования команд SHOW PROFILES;.

Исследуйте, какой engine используется в таблице БД test_db и приведите в ответе.

Измените engine и приведите время выполнения и запрос на изменения из профайлера в ответе:

* на MyISAM,
* на InnoDB.
```
mysql> SHOW PROFILES;
+----------+------------+------------------------------------+
| Query_ID | Duration   | Query                              |
+----------+------------+------------------------------------+
|        1 | 0.16029950 | ALTER TABLE orders ENGINE = MyISAM |
|        2 | 0.13943550 | ALTER TABLE orders ENGINE = InnoDB |
+----------+------------+------------------------------------+
2 rows in set, 1 warning (0.00 sec)
```

## Задача 4

### Изучите файл my.cnf в директории /etc/mysql.

Измените его согласно ТЗ (движок InnoDB):

* скорость IO важнее сохранности данных;
* нужна компрессия таблиц для экономии места на диске;
* размер буффера с незакомиченными транзакциями 1 Мб;
* буффер кеширования 30% от ОЗУ;
* размер файла логов операций 100 Мб.
* Приведите в ответе изменённый файл my.cnf.
```
bash-4.4# cat /etc/my.cnf
# For advice on how to change settings please see
# http://dev.mysql.com/doc/refman/8.0/en/server-configuration-defaults.html

[mysqld]
#
# Remove leading # and set to the amount of RAM for the most important data
# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
# innodb_buffer_pool_size = 128M
#
# Remove leading # to turn on a very important data integrity option: logging
# changes to the binary log between backups.
# log_bin
#
# Remove leading # to set options mainly useful for reporting servers.
# The server defaults are faster for transactions and fast SELECTs.
# Adjust sizes as needed, experiment to find the optimal values.
# join_buffer_size = 128M
# sort_buffer_size = 2M
# read_rnd_buffer_size = 2M

# Remove leading # to revert to previous value for default_authentication_plugin,
# this will increase compatibility with older clients. For background, see:
# https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_default_authentication_plugin
# default-authentication-plugin=mysql_native_password
skip-host-cache
skip-name-resolve
datadir=/var/lib/mysql
socket=/var/run/mysqld/mysqld.sock
secure-file-priv=/var/lib/mysql-files
user=mysql

pid-file=/var/run/mysqld/mysqld.pid

#Set IO Speed
innodb_flush_log_at_trx_commit = 0

#Set compression
innodb_file_format=Barracuda

#Set buffer
innodb_log_buffer_size  = 1M

#Set Cache size
key_buffer_size = 307М

#Set log size
max_binlog_size = 100M

[client]
socket=/var/run/mysqld/mysqld.sock

!includedir /etc/mysql/conf.d/
```

