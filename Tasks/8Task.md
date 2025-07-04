1. Вносим изменения в конфигурацию в соответствии с заданием.
```
sudo nano /etc/postgresql/15/main/postgresql.conf


log_lock_waits = on  
deadlock_timeout = 200ms
```
2. Поелс перезагрузки - пробуем воспроизвести ситуацию, при которой сессии заблокируют друг друга.
Для начала создаем таблицу и заполним ее тестовыми данными.
```
postgres=# create table testtable(test text);
CREATE TABLE

insert into testtable values ('test1');
INSERT 0 1
postgres=# insert into testtable values ('test2');
INSERT 0 1
```
3. В первой сессии открою транзакцию и заблокирую все записи.
```
postgres=# begin;
BEGIN

postgres=*# update testtable set test = 'S1';
UPDATE 2
```
4. Во второй - попробую обновить одну из заблокированных записей.
```
postgres=# begin;
BEGIN

postgres=*# update testtable set test = 'S2';
```
Транзакция зависает в ожидании разблокировки записи.
5. Коммитим изменения в первой сессии - вторая сессия сразу же разблокируется.
Смотрим логи.
```
2025-05-15 22:24:07.560 UTC [1741] postgres@postgres LOG:  process 1741 still waiting for ShareLock on transaction 749 after 200.622 ms
2025-05-15 22:24:07.560 UTC [1741] postgres@postgres DETAIL:  Process holding the lock: 1733. Wait queue: 1741.
2025-05-15 22:24:07.560 UTC [1741] postgres@postgres CONTEXT:  while updating tuple (0,9) in relation "testtable"
2025-05-15 22:24:07.560 UTC [1741] postgres@postgres STATEMENT:  update testtable set test = 'S2';
2025-05-15 22:24:28.211 UTC [1741] postgres@postgres LOG:  process 1741 acquired ShareLock on transaction 749 after 20851.363 ms
2025-05-15 22:24:28.211 UTC [1741] postgres@postgres CONTEXT:  while updating tuple (0,9) in relation "testtable"
2025-05-15 22:24:28.211 UTC [1741] postgres@postgres STATEMENT:  update testtable set test = 'S2';
```
Как видим, в логах пишется, что процесс ожидает разблокировки, а затем информаццию о том, что он получает разблокировку спустя определенное время.

6. Повторяем ситуацию и смотрим блокировки в pg_locks
```
postgres=# SELECT * FROM pg_locks pl LEFT JOIN pg_stat_activity psa ON pl.pid = psa.pid;
   locktype    | database | relation | page | tuple | virtualxid | transactionid | classid | objid | objsubid | virtualtransaction | pid
 |       mode       | granted | fastpath |          waitstart           | datid | datname  | pid  | leader_pid | usesysid | usename  | application_name | client_addr | client_hostname | client_port |         backend_start         |          xact_start           |          query_start          |         state_change          | wait_event_type |  wait_event   |        state        | backend_xid | backend_xmin
| query_id |                                     query                                     |  backend_type
---------------+----------+----------+------+-------+------------+---------------+---------+-------+----------+--------------------+------+------------------+---------+----------+------------------------------+-------+----------+------+------------+----------+----------+------------------+-------------+-----------------+-------------+-------------------------------+-------------------------------+-------------------------------+-------------------------------+-----------------+---------------+---------------------+-------------+--------------+----------+-------------------------------------------------------------------------------+----------------
 relation      |        5 |    24590 |      |       |            |               |         |       |          | 5/10               | 1828 | RowExclusiveLock | t       | t        |                              |     5 | postgres | 1828 |            |       10 | postgres | psql             |             |                 |          -1 | 2025-05-15 22:41:34.041082+00 | 2025-05-15 22:41:41.2782+00   | 2025-05-15 22:41:45.101973+00 | 2025-05-15 22:41:45.101975+00 | Lock            | transactionid | active              |         753 |          752
|          | update testtable set test = 'S2';                                             | client backend
 virtualxid    |          |          |      |       | 5/10       |               |         |       |          | 5/10               | 1828 | ExclusiveLock    | t       | t        |                              |     5 | postgres | 1828 |            |       10 | postgres | psql             |             |                 |          -1 | 2025-05-15 22:41:34.041082+00 | 2025-05-15 22:41:41.2782+00   | 2025-05-15 22:41:45.101973+00 | 2025-05-15 22:41:45.101975+00 | Lock            | transactionid | active              |         753 |          752
|          | update testtable set test = 'S2';                                             | client backend
 relation      |        5 |    12222 |      |       |            |               |         |       |          | 4/15               | 1741 | AccessShareLock  | t       | t        |                              |     5 | postgres | 1741 |            |       10 | postgres | psql             |             |                 |          -1 | 2025-05-15 22:23:57.422752+00 | 2025-05-15 22:41:56.998424+00 | 2025-05-15 22:41:56.998424+00 | 2025-05-15 22:41:56.998426+00 |                 |               | active              |             |          752
|          | SELECT * FROM pg_locks pl LEFT JOIN pg_stat_activity psa ON pl.pid = psa.pid; | client backend
 relation      |        5 |    12073 |      |       |            |               |         |       |          | 4/15               | 1741 | AccessShareLock  | t       | t        |                              |     5 | postgres | 1741 |            |       10 | postgres | psql             |             |                 |          -1 | 2025-05-15 22:23:57.422752+00 | 2025-05-15 22:41:56.998424+00 | 2025-05-15 22:41:56.998424+00 | 2025-05-15 22:41:56.998426+00 |                 |               | active              |             |          752
|          | SELECT * FROM pg_locks pl LEFT JOIN pg_stat_activity psa ON pl.pid = psa.pid; | client backend
 | ExclusiveLock    | t       | t        |                              |     5 | postgres | 1741 |            |       10 | postgres | psql             |             |                 |          -1 | 2025-05-15 22:23:57.422752+00 | 2025-05-15 22:41:56.998424+00 | 2025-06
-15 22:41:56.998424+00 | 2025-05-15 22:41:56.998426+00 |                 |               | active              |             |          752 |          | SELECT * FROM pg_locks pl LEFT JOIN pg_stat_activity psa ON pl.pid = psa.pid; | client backend
 relation      |        5 |    24590 |      |       |            |               |         |       |          | 3/76               | 1796 | RowExclusiveLock | t       | t        |                              |     5 | postgres | 1796 |            |       10 | postgres |
 psql             |             |                 |          -1 | 2025-05-15 22:40:32.457445+00 | 2025-05-15 22:41:03.040312+00 | 2025-05-15 22:41:04.24002+00  | 2025-05-15 22:41:04.240158+00 | Client          | ClientRead    | idle in transaction |         752 |
      |          | update testtable set test = 'S1';                                             | client backend
 virtualxid    |          |          |      |       | 3/76       |               |         |       |          | 3/76               | 1796 | ExclusiveLock    | t       | t        |                              |     5 | postgres | 1796 |            |       10 | postgres |
 psql             |             |                 |          -1 | 2025-05-15 22:40:32.457445+00 | 2025-05-15 22:41:03.040312+00 | 2025-05-15 22:41:04.24002+00  | 2025-05-15 22:41:04.240158+00 | Client          | ClientRead    | idle in transaction |         752 |
      |          | update testtable set test = 'S1';                                             | client backend
 relation      |        0 |     1260 |      |       |            |               |         |       |          | 4/15               | 1741 | AccessShareLock  | t       | f        |                              |     5 | postgres | 1741 |            |       10 | postgres |
 psql             |             |                 |          -1 | 2025-05-15 22:23:57.422752+00 | 2025-05-15 22:41:56.998424+00 | 2025-05-15 22:41:56.998424+00 | 2025-05-15 22:41:56.998426+00 |                 |               | active              |             |
  752 |          | SELECT * FROM pg_locks pl LEFT JOIN pg_stat_activity psa ON pl.pid = psa.pid; | client backend
 relation      |        0 |     1262 |      |       |            |               |         |       |          | 4/15               | 1741 | AccessShareLock  | t       | f        |                              |     5 | postgres | 1741 |            |       10 | postgres |
 psql             |             |                 |          -1 | 2025-05-15 22:23:57.422752+00 | 2025-05-15 22:41:56.998424+00 | 2025-05-15 22:41:56.998424+00 | 2025-05-15 22:41:56.998426+00 |                 |               | active              |             |
  752 |          | SELECT * FROM pg_locks pl LEFT JOIN pg_stat_activity psa ON pl.pid = psa.pid; | client backend
 transactionid |          |          |      |       |            |           752 |         |       |          | 5/10               | 1828 | ShareLock        | f       | f        | 2025-05-15 22:41:45.10238+00 |     5 | postgres | 1828 |            |       10 | postgres |
 psql             |             |                 |          -1 | 2025-05-15 22:41:34.041082+00 | 2025-05-15 22:41:41.2782+00   | 2025-05-15 22:41:45.101973+00 | 2025-05-15 22:41:45.101975+00 | Lock            | transactionid | active              |         753 |
  752 |          | update testtable set test = 'S2';                                             | client backend
 relation      |        0 |     2671 |      |       |            |               |         |       |          | 4/15               | 1741 | AccessShareLock  | t       | f        |                              |     5 | postgres | 1741 |            |       10 | postgres |
 psql             |             |                 |          -1 | 2025-05-15 22:23:57.422752+00 | 2025-05-15 22:41:56.998424+00 | 2025-05-15 22:41:56.998424+00 | 2025-05-15 22:41:56.998426+00 |                 |               | active              |             |
  752 |          | SELECT * FROM pg_locks pl LEFT JOIN pg_stat_activity psa ON pl.pid = psa.pid; | client backend
 relation      |        0 |     2676 |      |       |            |               |         |       |          | 4/15               | 1741 | AccessShareLock  | t       | f        |                              |     5 | postgres | 1741 |            |       10 | postgres |
 psql             |             |                 |          -1 | 2025-05-15 22:23:57.422752+00 | 2025-05-15 22:41:56.998424+00 | 2025-05-15 22:41:56.998424+00 | 2025-05-15 22:41:56.998426+00 |                 |               | active              |             |
  752 |          | SELECT * FROM pg_locks pl LEFT JOIN pg_stat_activity psa ON pl.pid = psa.pid; | client backend
 tuple         |        5 |    24590 |    0 |    17 |            |               |         |       |          | 5/10               | 1828 | ExclusiveLock    | t       | f        |                              |     5 | postgres | 1828 |            |       10 | postgres |
 psql             |             |                 |          -1 | 2025-05-15 22:41:34.041082+00 | 2025-05-15 22:41:41.2782+00   | 2025-05-15 22:41:45.101973+00 | 2025-05-15 22:41:45.101975+00 | Lock            | transactionid | active              |         753 |
  752 |          | update testtable set test = 'S2';                                             | client backend
 transactionid |          |          |      |       |            |           753 |         |       |          | 5/10               | 1828 | ExclusiveLock    | t       | f        |                              |     5 | postgres | 1828 |            |       10 | postgres |
 psql             |             |                 |          -1 | 2025-05-15 22:41:34.041082+00 | 2025-05-15 22:41:41.2782+00   | 2025-05-15 22:41:45.101973+00 | 2025-05-15 22:41:45.101975+00 | Lock            | transactionid | active              |         753 |
  752 |          | update testtable set test = 'S2';                                             | client backend
 transactionid |          |          |      |       |            |           752 |         |       |          | 3/76               | 1796 | ExclusiveLock    | t       | f        |                              |     5 | postgres | 1796 |            |       10 | postgres |
 psql             |             |                 |          -1 | 2025-05-15 22:40:32.457445+00 | 2025-05-15 22:41:03.040312+00 | 2025-05-15 22:41:04.24002+00  | 2025-05-15 22:41:04.240158+00 | Client          | ClientRead    | idle in transaction |         752 |
      |          | update testtable set test = 'S1';                                             | client backend
 relation      |        0 |     2672 |      |       |            |               |         |       |          | 4/15               | 1741 | AccessShareLock  | t       | f        |                              |     5 | postgres | 1741 |            |       10 | postgres |
 psql             |             |                 |          -1 | 2025-05-15 22:23:57.422752+00 | 2025-05-15 22:41:56.998424+00 | 2025-05-15 22:41:56.998424+00 | 2025-05-15 22:41:56.998426+00 |                 |               | active              |             |
  752 |          | SELECT * FROM pg_locks pl LEFT JOIN pg_stat_activity psa ON pl.pid = psa.pid; | client backend
 relation      |        0 |     2677 |      |       |            |               |         |       |          | 4/15               | 1741 | AccessShareLock  | t       | f        |                              |     5 | postgres | 1741 |            |       10 | postgres |
 psql             |             |                 |          -1 | 2025-05-15 22:23:57.422752+00 | 2025-05-15 22:41:56.998424+00 | 2025-05-15 22:41:56.998424+00 | 2025-05-15 22:41:56.998426+00 |                 |               | active              |             |
  752 |          | SELECT * FROM pg_locks pl LEFT JOIN pg_stat_activity psa ON pl.pid = psa.pid; | client backend
(17 rows)
```
Из интересного нам тут:
* RowExclusiveLock - стандартная блокировка при обновлении.
* ExclusiveLock - блокировка в рамках транзакции.
* ShareLock - ожидание доступности записи.


7. Взаимоблокировки - для воспроизведения данной ситуации нам в первую очередь нужно создать и заполнить  тестовую таблицу данными - таблица должна содержать ID и какие-либо другие данные, например так.
```
postgres=# select * from Test2;
 id | test
----+-------
  1 | test1
  2 | test2
  3 | test3
(3 rows)
```
8. Откроем три сессии.
	1. В первой сессии - поменяем значение Test1 (1).
	```
	postgres=# begin;
	BEGIN
	postgres=*# update Test2 set test = 'TestS1' where Id=1;
	UPDATE 1
	```
	2. Во второй  Test2 (2).
	```
	postgres=# begin;
	BEGIN
	postgres=*# update Test2 set test = 'TestS2' where Id=2;
	UPDATE 1
	```
	3. Во второй  Test2 (3).
	```
	postgres=# begin;
	BEGIN
	postgres=*# update Test2 set test = 'TestS3' where Id=3;
	UPDATE 1
	```
9. Создадим взаимную блокировку.
	1. В первой сессии - обновим значение 2 (заблокированное другой транзакцией) - по результатам ожидаемо зависнем в ожиании.
	```
	UPDATE Test2 SET test = test || 'S1' WHERE id = 2;
	```
	2. Во второй сессии сделаем то-же самое, но с записью 3.
	```
	UPDATE Test2 SET test = test || 'S2' WHERE id = 3;
	```
	3. В третьей сделаем тоже самое, но с записью 1.
	```
	UPDATE Test2 SET test = test || 'S3' WHERE id = 1;
	ERROR:  deadlock detected
	DETAIL:  Process 1796 waits for ShareLock on transaction 764; blocked by process 1942.
	Process 1942 waits for ShareLock on transaction 765; blocked by process 1741.
	Process 1741 waits for ShareLock on transaction 766; blocked by process 1796.
	HINT:  See server log for query details.
	CONTEXT:  while updating tuple (0,1) in relation "test2"
	```
     В третьей сессии мы получаем взаимную блокировку - прошще говоря Deadlock - PostgreeSQL - убил третью сессию, чтобы разрешить конфликт и разрешить другим сессиям создать блокировку.