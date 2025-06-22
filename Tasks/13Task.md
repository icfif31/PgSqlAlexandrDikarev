1. Для начала подготовим три инстанса VM (далее VM1, VM2, VM3) - настроим у виртуалок две сети - одну в рамках NAT, а вторую внутреннюю между виртуалками, дав каждой из них статический IP (192.168.10.10, 192.168.10.20, 192.168.10.30) во внутренней сети. (На самом деле - это был самый сложный пункт - ввиду того, что я не сетевик).
1. Производим установку на Postgreesql 15 на всех машинах:
```
sudo apt install -y postgresql-common
sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh

sudo apt update
sudo apt install postgresql-15
```
3. Настраиваем PGSql.
`sudo nano /etc/postgresql/15/main/postgresql.conf`
```
listen_addresses = '*'
wal_level = logical
max_replication_slots = 10
max_wal_senders = 10
```
`sudo nano /etc/postgresql/15/main/pg_hba.conf`
```
host    all             all             192.168.10.0/24         scram-sha-256
host    replication     all             192.168.10.0/24         scram-sha-256
```
`sudo systemctl restart postgresql`
4. Создаем пользователя и бд на всех виртуалках.
```
sudo -u postgres psql -c "CREATE USER replicator WITH REPLICATION ENCRYPTED PASSWORD 'replica123';"
CREATE ROLE
sudo -u postgres psql -c "CREATE DATABASE db_replication;"
CREATE DATABASE
```
5. Производим настройки на VM1 - создаем таблицы и публикуем таблицу `test`.
```
sudo -u postgres psql -d db_replication

CREATE TABLE test (id SERIAL PRIMARY KEY, data TEXT);
CREATE TABLE test2 (id SERIAL PRIMARY KEY, data TEXT);

CREATE PUBLICATION test_pub FOR TABLE test;
```
6. Аналогичные настройки проводим на VM2 - но публикуем `test2`.
```
sudo -u postgres psql -d db_replication 

CREATE TABLE test2 (id SERIAL PRIMARY KEY, data TEXT);
CREATE TABLE test (id SERIAL PRIMARY KEY, data TEXT);

CREATE PUBLICATION test2_pub FOR TABLE test2;
```
7. Возвращаемся на VM1 и настраиваем подписку на test2.
```
db_replication=# CREATE SUBSCRIPTION test2_sub
CONNECTION 'host=192.168.10.20 port=5432 user=replicator password=replica123 dbname=db_replication'
PUBLICATION test2_pub;
NOTICE:  created replication slot "test2_sub" on publisher
CREATE SUBSCRIPTION
```
8. Аналогично на VM2 - настраиваем подписку на test.
```
db_replication=# CREATE SUBSCRIPTION test_sub
CONNECTION 'host=192.168.10.10 port=5432 user=replicator password=replica123 dbname=db_replication'
PUBLICATION test_pub;
NOTICE:  created replication slot "test_sub" on publisher
CREATE SUBSCRIPTION
```
9. Осталось настроить только VM3 для работы бекапов обоих таблиц.
Для начала создадим таблицы.
```
CREATE TABLE test (id SERIAL PRIMARY KEY, data TEXT);
CREATE TABLE test2 (id SERIAL PRIMARY KEY, data TEXT);
```
10. Затем настроим подписки.
```
CREATE SUBSCRIPTION test1_sub 
CONNECTION 'host=192.168.10.10 port=5432 user=replicator password=replica123 dbname=db_replication' 
PUBLICATION test_pub;

CREATE SUBSCRIPTION test2_sub1
CONNECTION 'host=192.168.10.20 port=5432 user=replicator password=replica123 dbname=db_replication' 
PUBLICATION test2_pub;
```
11. При проверке - оказалось, что репликация не работает - причина оказалась в правах - выдал права пользователю.
```
GRANT SELECT ON TABLE test TO replicator;
GRANT SELECT ON TABLE test2 TO replicator;
```
12. Проверяем - создаем на VM1 запись в таблице Test.
```
INSERT INTO test (data) VALUES ('Data from VM1');
INSERT 0 1
```
13. Проверяем наличие данных на VM2 и VM3 (аналогично).
```
db_replication=# SELECT * FROM test;
 id |     data
----+---------------
  6 | Data from VM1
(1 row)
```
14. Проводим такую же операцию с test2 на VM2.
```
INSERT INTO test2 (data) VALUES ('Data from VM2');
```
15. Проверяем на VM1 и VM3 (аналогично).
```
db_replication=# SELECT * FROM test2;
 id |     data
----+---------------
  2 | Data from VM2
```

Как видно - рекликация полностью работает.