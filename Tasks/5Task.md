1. Подключаю репозиторий Postgresql

```
sudo apt install -y postgresql-common
sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh
```

2. Устанавливаю 15 версию postgresql.

```
sudo apt update
sudo apt install postgresql-15
```

3. Проверяю, что кластер запущен.

```
sudo -u postgres pg_lsclusters

Ver Cluster Port Status Owner    Data directory              Log file
15  main    5433 online postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log

```

4. Открываю конфигурацию:
   `sudo nano /etc/postgresql/15/main/postgresql.conf`  
   Урезаю функционал, меняю настройки базы с целью повышения производительности - невзирая на его нужность и важность для реальной работы.

```
# Настраиваем потребление памяти.
shared_buffers = 2GB
work_mem = 64MB
wal_buffers = 16MB
max_wal_size = 4GB

# Отключаем все что нам не нужно для тестов и красивых цифр, невзирая на важность этих функций.
synchronous_commit = off
full_page_writes = off
checkpoint_timeout = 30min
logging_collector = off
```

5. Захожу от пользователя postgres, произвожу иницилизацию pgbench.

```
pgbench -i

dropping old tables...
NOTICE:  table "pgbench_accounts" does not exist, skipping
NOTICE:  table "pgbench_branches" does not exist, skipping
NOTICE:  table "pgbench_history" does not exist, skipping
NOTICE:  table "pgbench_tellers" does not exist, skipping
creating tables...
generating data (client-side)...
100000 of 100000 tuples (100%) done (elapsed 0.11 s, remaining 0.00 s)
vacuuming...
creating primary keys...
done in 0.41 s (drop tables 0.01 s, create tables 0.03 s, client-side generate 0.16 s, vacuum 0.07 s, primary keys 0.15 s).
```

6. Запускаю и поулчаю следующий результат - TPS 317 - неплохо.

```
pgbench

pgbench (15.12 (Ubuntu 15.12-1.pgdg24.04+1))
starting vacuum...end.
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 1
number of threads: 1
maximum number of tries: 1
number of transactions per client: 10
number of transactions actually processed: 10/10
number of failed transactions: 0 (0.000%)
latency average = 3.145 ms
initial connection time = 8.746 ms
tps = 317.924588 (without initial connection time)
```
