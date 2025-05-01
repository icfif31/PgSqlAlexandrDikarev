1. Cоздал кластер PostgresSQL15, зашел в psql из под юзера postgres.
```
sudo -u postgres psql
```
2. Создал базу данных testdb и сделал её активной.
```
postgres=# CREATE DATABASE testdb;
CREATE DATABASE
postgres=# \c testdb
You are now connected to database "testdb" as user "postgres".
```
3. Создал схему testnm и проверил что она создалась.
```
testdb=# CREATE SCHEMA testnm;
CREATE SCHEMA

testdb=# \dn
      List of schemas
  Name  |       Owner
--------+-------------------
 public | pg_database_owner
 testnm | postgres
```
4. Создал таблицу t1 с полем с1 типа int, выставил значение 1 в поле c1.
Вывел данную таблицу.
```
testdb=# CREATE TABLE testnm.t1 (
testdb(#     c1 INTEGER
testdb(# );
CREATE TABLE
testdb=# INSERT INTO testnm.t1 (c1) VALUES (1);
INSERT 0 1
testdb=#
testdb=# SELECT * FROM testnm.t1;
 c1
----
  1
(1 row)
```
5. Создал новую роль readonly и дал ей право на подключение к бд testdb.
```
testdb=# CREATE ROLE readonly;
CREATE ROLE
testdb=# GRANT CONNECT ON DATABASE testdb TO readonly;
GRANT
```
6. Выдал права на использование схемы testnm и на select. 
```
testdb=# grant usage on SCHEMA testnm to readonly;
GRANT
testdb=# grant SELECT on all TABLEs in SCHEMA testnm TO readonly;
GRANT
```
7. Создал пользователя testread с паролем test123 и дал ему роль readonly.
```
testdb=# CREATE USER testread WITH PASSWORD 'test123';
CREATE ROLE
testdb=# GRANT readonly TO testread;
GRANT ROLE
```
8. Попытался подключиться под `testread` и выполнить `SELECT * FROM t1;`
```
ailed for user "testread"
Previous connection kept
postgres=# \q
alexandr@alexandrvm:~$ psql -h 127.0.0.1 -U testread -d testdb -W
Password:
psql (15.12 (Ubuntu 15.12-1.pgdg24.04+1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off)
Type "help" for help.


testdb=> Select * From t1;
ERROR:  relation "t1" does not exist
```
9. **Причина**: Таблица `t1` создана в схеме `testnm`, а не в `public`. По умолчанию PostgreSQL ищет таблицы в схемах, указанных в `search_path` (обычно `"$user", public`).
**Решение**: Указать схему явно
10. Просмотрел список таблиц. 
```
testdb=> \dt
Did not find any relations.
```
В схеме "по умолчанию" - public таблиц действительно нет, чтобы увидеть наши таблицы - дополним запрос названием схемы:
```
testdb=> \dt testnm.*
        List of relations
 Schema | Name | Type  |  Owner
--------+------+-------+----------
 testnm | t1   | table | postgres
(1 row)
```
11. В данном случае таблица исходно была создана в схеме, иначе бы нам пришлось удалять её. для этого необходимо вернуться  в testbd под postgres
```
alexandr@alexandrvm:~$ sudo -u postgres psql
[sudo] password for alexandr:
psql (15.12 (Ubuntu 15.12-1.pgdg24.04+1))
Type "help" for help.

postgres=# \c testdb
You are now connected to database "testdb" as user "postgres".
```
Затем выполнить команду ` DROP TABLE t1;`, после чего создать ее в нужной схеме.
12. Выполнил `select * from testnm.t1;`
```
testdb=> select * from testnm.t1;
 c1
----
  1
(1 row)
```
13. Попробовал создать таблицу t2 и заполнить ее значениями - на удивление - это получилось
```
testdb=> create table t2(c1 integer); insert into t2 values (2);
CREATE TABLE
INSERT 0 1
```
14. Дело в том, что в версии 14 - "по-умолчанию" всем пользователям доступно создание таблиц в схеме public. Чтобы запретить это - нам нужно запретить соответствующие действия в данной схеме.
```
\c testdb postgres
REVOKE CREATE on SCHEMA public FROM public;
```
15. Теперь подключаюсь к пользователю с ролью readonly и пробую создать таблицу. Ожидаемо получаю ошибку.
```
\c testdb testread

create table t3(c1 int);
ERROR:  permission denied for schema public
LINE 1: create table t3(c1 int);
```