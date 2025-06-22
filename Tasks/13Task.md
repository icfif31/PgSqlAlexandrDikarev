1. Для начала предлагаю создать сеть в докере - в рамках которой будут подняты три инстанса постгреса.
```
docker network create pg-net

docker run -d --name pg1 --network pg-net -p 5432:5432 -e POSTGRES_PASSWORD=postgres postgres:15
docker run -d --name pg2 --network pg-net -p 5433:5432 -e POSTGRES_PASSWORD=postgres postgres:15
docker run -d --name pg3 --network pg-net -p 5434:5432 -e POSTGRES_PASSWORD=postgres postgres:15
```
2. Производим базовые настройки в каждом из контейнеров.
```
wal_level = 'logical';
max_replication_slots = 10;
ALTER SYSTEM SET max_wal_senders = 10;
```
3. Добавляем в pg_hba.conf следующие настройки:
```
host all all 0.0.0.0/0 scram-sha-256
host replication replicator 0.0.0.0/0 scram-sha-256
```
4. Производим настройки на pg1 - создаем пользователя, базу, таблицы и настраиваем публикацию таблицы test.
```
CREATE USER replicator WITH REPLICATION ENCRYPTED PASSWORD 'replica123';
CREATE DATABASE db_replication;

\c db_replication

CREATE TABLE test (id SERIAL PRIMARY KEY, data TEXT);
CREATE TABLE test2 (id SERIAL PRIMARY KEY, data TEXT);

CREATE PUBLICATION test_pub FOR TABLE test;
```
5. Почти тоже самое делаем в pg2, только публикуем table2.
```

```
1. 
