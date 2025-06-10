1. Создает VM с 2 ядрами и 4 гигабайтами ОЗУ.
Устанавливаем на него PostgreSQL 15 с дефолтными настройками.
```
sudo apt install postgresql-15
```
2. Переключаюсь на пользователя postgres и инициализирую бд для тестов.
```
sudo su postgres
pgbench -i postgres

postgres@ubuntu-vm:/home/alexandr$ pgbench -i postgres
dropping old tables...
NOTICE:  table "pgbench_accounts" does not exist, skipping
NOTICE:  table "pgbench_branches" does not exist, skipping
NOTICE:  table "pgbench_history" does not exist, skipping
NOTICE:  table "pgbench_tellers" does not exist, skipping
creating tables...
generating data (client-side)...
100000 of 100000 tuples (100%) done (elapsed 0.23 s, remaining 0.00 s)
vacuuming...
creating primary keys...
done in 0.74 s (drop tables 0.01 s, create tables 0.05 s, client-side generate 0.32 s, vacuum 0.27 s, primary keys 0.10 s).
```
3. Запускаем тест
```
postgres@ubuntu-vm:/home/alexandr$ pgbench -c8 -P 6 -T 60 -U postgres postgres
pgbench (15.12 (Ubuntu 15.12-1.pgdg24.04+1))
starting vacuum...end.
progress: 6.0 s, 368.8 tps, lat 21.405 ms stddev 15.163, 0 failed
progress: 12.0 s, 372.3 tps, lat 21.397 ms stddev 15.153, 0 failed
progress: 18.0 s, 374.5 tps, lat 21.429 ms stddev 15.119, 0 failed
progress: 24.0 s, 364.7 tps, lat 21.939 ms stddev 15.425, 0 failed
progress: 30.0 s, 358.7 tps, lat 22.266 ms stddev 15.963, 0 failed
progress: 36.0 s, 348.5 tps, lat 22.930 ms stddev 18.520, 0 failed
progress: 42.0 s, 237.7 tps, lat 33.645 ms stddev 27.672, 0 failed
progress: 48.0 s, 334.6 tps, lat 23.909 ms stddev 19.182, 0 failed
progress: 54.0 s, 351.2 tps, lat 22.742 ms stddev 16.760, 0 failed
progress: 60.0 s, 259.0 tps, lat 30.903 ms stddev 34.866, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 20228
number of failed transactions: 0 (0.000%)
latency average = 23.702 ms
latency stddev = 19.774 ms
initial connection time = 60.185 ms
tps = 337.245709 (without initial connection time)
postgres@ubuntu-vm:/home/alexandr$
```
4. Меняем настройки в файле /etc/postgresql/15/main/postgresql.conf на настройки предложенные в однои из файлов.

	Повторно запускаем тест:
	```
	postgres@ubuntu-vm:/home/alexandr$ pgbench -c8 -P 6 -T 60 -U postgres postgres
	pgbench (15.12 (Ubuntu 15.12-1.pgdg24.04+1))
	starting vacuum...end.
	progress: 6.0 s, 338.2 tps, lat 23.423 ms stddev 15.902, 0 failed
	progress: 12.0 s, 373.3 tps, lat 21.412 ms stddev 15.199, 0 failed
	progress: 18.0 s, 373.2 tps, lat 21.440 ms stddev 15.950, 0 failed
	progress: 24.0 s, 374.0 tps, lat 21.375 ms stddev 15.288, 0 failed
	progress: 30.0 s, 378.0 tps, lat 21.146 ms stddev 15.024, 0 failed
	progress: 36.0 s, 368.2 tps, lat 21.733 ms stddev 16.687, 0 failed
	progress: 42.0 s, 371.7 tps, lat 21.499 ms stddev 14.749, 0 failed
	progress: 48.0 s, 374.5 tps, lat 21.338 ms stddev 15.660, 0 failed
	progress: 54.0 s, 371.5 tps, lat 21.553 ms stddev 14.969, 0 failed
	progress: 60.0 s, 374.5 tps, lat 21.325 ms stddev 16.445, 0 failed
	transaction type: <builtin: TPC-B (sort of)>
	scaling factor: 1
	query mode: simple
	number of clients: 8
	number of threads: 1
	maximum number of tries: 1
	duration: 60 s
	number of transactions actually processed: 22190
	number of failed transactions: 0 (0.000%)
	latency average = 21.613 ms
	latency stddev = 15.610 ms
	initial connection time = 42.688 ms
	tps = 369.886720 (without initial connection time)
	postgres@ubuntu-vm:/home/alexandr$
	```

Как видим - результат TPS, Среднего времени отклика(latency average) и latency stddev стал стал чуть лучше за счет выделения большего объема памяти и большего объема кешей.

5. Созддим Бд и таблицу с одним текстовым полем. Заполним их сгенерированными данными в размере 1_000_000 строк.
```
postgres=# create database testdb;
CREATE DATABASE
postgres=# \c testdb
psql (17.4 (Ubuntu 17.4-1.pgdg24.04+2), server 15.12 (Ubuntu 15.12-1.pgdg24.04+1))
You are now connected to database "testdb" as user "postgres".
testdb=# CREATE TABLE student(
testdb(#   id serial,
testdb(#   fio char(100)
testdb(# ) WITH (autovacuum_enabled = off);
CREATE TABLE
testdb=#
testdb=# INSERT INTO student(fio) SELECT 'noname' FROM generate_series(1,1000000);
INSERT 0 1000000
testdb=#
```
6. Посмотрим размер созданной таблицы.
```
testdb=# SELECT pg_size_pretty(pg_total_relation_size('student'));
 pg_size_pretty
----------------
 135 MB
(1 row)
```
7. Обновим 5 раз таблицу Student, добавив символ X.
```
UPDATE student SET FIO = FIO || 'X';
UPDATE student SET FIO = FIO || 'X';
UPDATE student SET FIO = FIO || 'X';
UPDATE student SET FIO = FIO || 'X';
UPDATE student SET FIO = FIO || 'X';
```
8. Повторно проверим размерп файла с таблицей.
```
testdb=# SELECT pg_size_pretty(pg_total_relation_size('student'));
 pg_size_pretty
----------------
 808 MB
(1 row)
```
9. Отключаем автовакум для таблицы Student:
```
testdb=# ALTER TABLE student SET (autovacuum_enabled = false);
ALTER TABLE
```
10. Обновляем записи в таблице 10 раз и смотрим размер таблицы.
```
testdb=# UPDATE student SET FIO = FIO || 'X';
UPDATE 1000000
testdb=# UPDATE student SET FIO = FIO || 'X';
UPDATE 1000000
testdb=# UPDATE student SET FIO = FIO || 'X';
UPDATE 1000000
testdb=# UPDATE student SET FIO = FIO || 'X';
UPDATE 1000000
testdb=# UPDATE student SET FIO = FIO || 'X';
UPDATE 1000000
testdb=# UPDATE student SET FIO = FIO || 'X';
UPDATE 1000000
testdb=# UPDATE student SET FIO = FIO || 'X';
UPDATE 1000000
testdb=# UPDATE student SET FIO = FIO || 'X';
UPDATE 1000000
testdb=# UPDATE student SET FIO = FIO || 'X';
UPDATE 1000000
testdb=# UPDATE student SET FIO = FIO || 'X';
UPDATE 1000000
testdb=# SELECT pg_size_pretty(pg_total_relation_size('student'));
 pg_size_pretty
----------------
 2156 MB
(1 row)
```
Без включенного автовакума размер используемого буфера многократно возрастает.

