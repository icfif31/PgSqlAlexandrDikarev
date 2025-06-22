1. Начнем с того, что создадим тестовую бд.
```
CREATE DATABASE backup_demo;
CREATE DATABASE
```
2. Переключимся на нее.
```
postgres=# \c backup_demo
psql (17.4 (Ubuntu 17.4-1.pgdg24.04+2), server 15.12 (Ubuntu 15.12-1.pgdg24.04+1))
You are now connected to database "backup_demo" as user "postgres".
```
3. Создадим таблицу.
```
backup_demo=# CREATE TABLE table1 (
    id SERIAL PRIMARY KEY,
    data TEXT,
    created_at TIMESTAMP DEFAULT NOW()
);
CREATE TABLE
```
4. Заполним ее сгенированными данными.
```
backup_demo=# INSERT INTO hw_schema.table1 (data)
SELECT
    md5(random()::text)
FROM generate_series(1, 100);
```
**COPY**

5. Сделаем логический бекап в формат CSV.
```
backup_demo=# COPY table1
TO '/var/lib/postgresql/backups/table1_backup.csv'
WITH (FORMAT CSV, HEADER);
COPY 100
```
6. В сгенерированном CSV файле содержится дамп данных, разделенный запятыми.
```
alexandr@ubuntu-vm:~$ cat /var/lib/postgresql/backups/table1_backup.csv
id,data,created_at
1,8450984f13e983c02184b8e9318aef53,2025-06-22 15:39:23.754073
2,905fd17e22bb6352f7bdc720f102303d,2025-06-22 15:39:23.754073
3,053f0f6e1ff766b42d1c3d95755eb31c,2025-06-22 15:39:23.754073
4,6743078634ecedaf65c7a4a9544c9f32,2025-06-22 15:39:23.754073
5,cf712a4f9dd5813ec7fff2c8cc7160af,2025-06-22 15:39:23.754073
6,13c7e584ab35c10efa6146fc97f04c3a,2025-06-22 15:39:23.754073
7,4cd1223da91cd47fe0a432d9bd0a2de0,2025-06-22 15:39:23.754073
8,7a0bb72a59f1bbae60857a9e2f4d7a23,2025-06-22 15:39:23.754073
...
```
7. Создадим таблицу2 - аналогичную по структуре первой, и развернем в нее наш бекап.
```
backup_demo=# CREATE TABLE table2 (
    id SERIAL PRIMARY KEY,
    data TEXT,
    created_at TIMESTAMP DEFAULT NOW()
);
CREATE TABLE

backup_demo=# COPY table2
FROM '/var/lib/postgresql/backups/table1_backup.csv'
WITH (FORMAT CSV, HEADER);
COPY 100
```
Как видно - данные были успешно перенесены в новую таблицу.

**pg_dump**

8. Теперь сделаем дамп через pg_dump - для этого в терминале выполним команду.
```
pg_dump -Fc -d backup_demo -t 'table1' -t 'table2' -U postgres -Fc -f /tmp/backup_demo.gz
```
9. Создадим новую БД, убедимся, что она пустая.
```
CREATE DATABASE restore_demo;
CREATE DATABASE

\c restore_demo
\dt

Did not find any relations.
```
10. Возвращаемся в терминал - выполняем восcтановление.
```
pg_restore -d restore_demo -t 'table2' -U postgres /tmp/backup_demo.gz
```
11. Возвращаемся в psql и проверяем, что данные на месте.
```
\c restore_demo

restore_demo=# \dt
         List of relations
 Schema |  Name  | Type  |  Owner
--------+--------+-------+----------
 public | table2 | table | postgres
(1 row)

restore_demo=# select count(*) from table2;
 count
-------
   100
(1 row)

restore_demo=# select * from table2;
 id  |               data               |         created_at
-----+----------------------------------+----------------------------
   1 | 8450984f13e983c02184b8e9318aef53 | 2025-06-22 15:39:23.754073
   2 | 905fd17e22bb6352f7bdc720f102303d | 2025-06-22 15:39:23.754073
   3 | 053f0f6e1ff766b42d1c3d95755eb31c | 2025-06-22 15:39:23.754073
   4 | 6743078634ecedaf65c7a4a9544c9f32 | 2025-06-22 15:39:23.754073
   5 | cf712a4f9dd5813ec7fff2c8cc7160af | 2025-06-22 15:39:23.754073
   6 | 13c7e584ab35c10efa6146fc97f04c3a | 2025-06-22 15:39:23.754073
   7 | 4cd1223da91cd47fe0a432d9bd0a2de0 | 2025-06-22 15:39:23.754073
   ....   
```
Как видно - данные были успешно востановлены.
