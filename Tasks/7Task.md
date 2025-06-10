1. Поднял VM и установил на нее PGSQl.
```
sudo apt install postgresql-15
```
2. Установил таймаут выполнения контрольной точки на 30 секунд.
```
alter system set checkpoint_timeout = '30s';  
ALTER SYSTEM
```
3. Фиксируем позицию WAL.
```
postgres=# SELECT pg_walfile_name(pg_current_wal_lsn()), pg_current_wal_lsn();
     pg_walfile_name      | pg_current_wal_lsn
--------------------------+--------------------
 000000010000000000000022 | 0/2227B008
(1 row)
```
4. Запускаем тест на 5 минут
```
pgbench -c8 -P 6 -T 600 -U postgres postgres
pgbench (15.12 (Ubuntu 15.12-1.pgdg24.04+1))
starting vacuum...end.
progress: 6.0 s, 335.1 tps, lat 23.628 ms stddev 16.655, 0 failed
progress: 12.0 s, 345.2 tps, lat 23.179 ms stddev 17.108, 0 failed
progress: 18.0 s, 348.8 tps, lat 22.919 ms stddev 16.571, 0 failed
progress: 24.0 s, 344.3 tps, lat 23.233 ms stddev 16.758, 0 failed
progress: 30.0 s, 341.3 tps, lat 23.413 ms stddev 17.176, 0 failed
progress: 36.0 s, 365.7 tps, lat 21.874 ms stddev 16.375, 0 failed
progress: 42.0 s, 364.3 tps, lat 21.937 ms stddev 16.729, 0 failed
progress: 48.0 s, 354.7 tps, lat 22.547 ms stddev 15.376, 0 failed
progress: 54.0 s, 346.8 tps, lat 23.059 ms stddev 16.745, 0 failed
progress: 60.0 s, 353.5 tps, lat 22.617 ms stddev 15.823, 0 failed
progress: 66.0 s, 349.5 tps, lat 22.859 ms stddev 17.584, 0 failed
progress: 72.0 s, 348.3 tps, lat 22.963 ms stddev 17.694, 0 failed
progress: 78.0 s, 341.0 tps, lat 23.418 ms stddev 16.671, 0 failed
progress: 84.0 s, 341.7 tps, lat 23.381 ms stddev 17.764, 0 failed
progress: 90.0 s, 351.7 tps, lat 22.793 ms stddev 16.554, 0 failed
progress: 96.0 s, 355.8 tps, lat 22.461 ms stddev 17.383, 0 failed
progress: 102.0 s, 341.2 tps, lat 23.451 ms stddev 16.797, 0 failed
progress: 108.0 s, 343.0 tps, lat 23.286 ms stddev 17.479, 0 failed
progress: 114.0 s, 355.8 tps, lat 22.503 ms stddev 16.881, 0 failed
progress: 120.0 s, 341.7 tps, lat 23.408 ms stddev 17.054, 0 failed
progress: 126.0 s, 349.1 tps, lat 22.884 ms stddev 17.246, 0 failed
progress: 132.0 s, 331.0 tps, lat 24.097 ms stddev 16.919, 0 failed
progress: 138.0 s, 358.5 tps, lat 22.331 ms stddev 15.009, 0 failed
progress: 144.0 s, 353.7 tps, lat 22.605 ms stddev 16.812, 0 failed
progress: 150.0 s, 357.2 tps, lat 22.393 ms stddev 17.080, 0 failed
progress: 156.0 s, 329.3 tps, lat 24.270 ms stddev 18.424, 0 failed
progress: 162.0 s, 356.2 tps, lat 22.466 ms stddev 16.445, 0 failed
progress: 168.0 s, 358.3 tps, lat 22.306 ms stddev 16.671, 0 failed
progress: 174.0 s, 362.5 tps, lat 22.023 ms stddev 16.299, 0 failed
progress: 180.0 s, 163.4 tps, lat 43.827 ms stddev 166.091, 0 failed
progress: 186.0 s, 69.2 tps, lat 127.000 ms stddev 214.904, 0 failed
progress: 192.0 s, 100.0 tps, lat 80.016 ms stddev 49.852, 0 failed
progress: 198.0 s, 103.4 tps, lat 77.407 ms stddev 47.470, 0 failed
progress: 204.0 s, 103.5 tps, lat 76.792 ms stddev 52.300, 0 failed
progress: 210.0 s, 103.2 tps, lat 77.554 ms stddev 61.187, 0 failed
progress: 216.0 s, 99.8 tps, lat 80.309 ms stddev 57.933, 0 failed
progress: 222.0 s, 99.4 tps, lat 79.959 ms stddev 53.132, 0 failed
progress: 228.0 s, 97.8 tps, lat 82.244 ms stddev 57.923, 0 failed
progress: 234.0 s, 98.2 tps, lat 81.203 ms stddev 55.709, 0 failed
progress: 240.0 s, 98.2 tps, lat 81.736 ms stddev 51.055, 0 failed
progress: 246.0 s, 97.0 tps, lat 82.266 ms stddev 56.386, 0 failed
progress: 252.0 s, 90.3 tps, lat 88.665 ms stddev 59.648, 0 failed
progress: 258.0 s, 98.5 tps, lat 81.172 ms stddev 54.116, 0 failed
progress: 264.0 s, 96.5 tps, lat 82.907 ms stddev 57.610, 0 failed
progress: 270.0 s, 98.9 tps, lat 80.598 ms stddev 52.504, 0 failed
progress: 276.0 s, 97.6 tps, lat 81.987 ms stddev 52.085, 0 failed
progress: 282.0 s, 101.0 tps, lat 79.020 ms stddev 51.316, 0 failed
progress: 288.0 s, 97.8 tps, lat 81.687 ms stddev 51.757, 0 failed
progress: 294.0 s, 99.5 tps, lat 80.309 ms stddev 50.503, 0 failed
progress: 300.0 s, 101.3 tps, lat 78.376 ms stddev 59.610, 0 failed
progress: 306.0 s, 100.8 tps, lat 79.593 ms stddev 56.250, 0 failed
progress: 312.0 s, 100.0 tps, lat 79.607 ms stddev 55.129, 0 failed
progress: 318.0 s, 77.3 tps, lat 103.899 ms stddev 83.764, 0 failed
progress: 324.0 s, 103.5 tps, lat 77.377 ms stddev 52.207, 0 failed
progress: 330.0 s, 98.5 tps, lat 80.588 ms stddev 50.415, 0 failed
progress: 336.0 s, 97.8 tps, lat 82.103 ms stddev 52.821, 0 failed
progress: 342.0 s, 104.3 tps, lat 76.644 ms stddev 48.307, 0 failed
progress: 348.0 s, 101.2 tps, lat 79.162 ms stddev 48.672, 0 failed
progress: 354.0 s, 102.3 tps, lat 77.658 ms stddev 52.979, 0 failed
progress: 360.0 s, 99.2 tps, lat 80.917 ms stddev 57.396, 0 failed
progress: 366.0 s, 97.3 tps, lat 81.889 ms stddev 59.046, 0 failed
progress: 372.0 s, 98.6 tps, lat 81.235 ms stddev 62.361, 0 failed
progress: 378.0 s, 99.2 tps, lat 80.466 ms stddev 52.778, 0 failed
progress: 384.0 s, 95.3 tps, lat 84.050 ms stddev 64.296, 0 failed
progress: 390.0 s, 104.2 tps, lat 76.594 ms stddev 51.192, 0 failed
progress: 396.0 s, 103.5 tps, lat 76.878 ms stddev 49.880, 0 failed
progress: 402.0 s, 101.8 tps, lat 78.973 ms stddev 56.961, 0 failed
progress: 408.0 s, 107.3 tps, lat 74.444 ms stddev 46.797, 0 failed
progress: 414.0 s, 107.8 tps, lat 74.348 ms stddev 46.967, 0 failed
progress: 420.0 s, 107.7 tps, lat 74.139 ms stddev 52.913, 0 failed
progress: 426.0 s, 105.6 tps, lat 75.749 ms stddev 52.349, 0 failed
progress: 432.0 s, 103.2 tps, lat 77.388 ms stddev 51.902, 0 failed
progress: 438.0 s, 103.3 tps, lat 77.501 ms stddev 50.670, 0 failed
progress: 444.0 s, 105.5 tps, lat 75.705 ms stddev 46.919, 0 failed
progress: 450.0 s, 103.3 tps, lat 77.207 ms stddev 50.500, 0 failed
progress: 456.0 s, 102.7 tps, lat 77.703 ms stddev 49.419, 0 failed
progress: 462.0 s, 96.7 tps, lat 82.480 ms stddev 54.177, 0 failed
progress: 468.0 s, 97.5 tps, lat 82.115 ms stddev 51.466, 0 failed
progress: 474.0 s, 97.7 tps, lat 81.968 ms stddev 53.204, 0 failed
progress: 480.0 s, 99.0 tps, lat 80.786 ms stddev 51.253, 0 failed
progress: 486.0 s, 101.3 tps, lat 78.838 ms stddev 47.072, 0 failed
progress: 492.0 s, 99.9 tps, lat 80.073 ms stddev 52.118, 0 failed
progress: 498.0 s, 100.0 tps, lat 79.779 ms stddev 50.963, 0 failed
progress: 504.0 s, 97.8 tps, lat 81.811 ms stddev 51.128, 0 failed
progress: 510.0 s, 99.3 tps, lat 80.632 ms stddev 48.785, 0 failed
progress: 516.0 s, 96.8 tps, lat 82.707 ms stddev 54.699, 0 failed
progress: 522.0 s, 101.0 tps, lat 79.110 ms stddev 49.585, 0 failed
progress: 528.0 s, 103.5 tps, lat 77.145 ms stddev 51.822, 0 failed
progress: 534.0 s, 102.0 tps, lat 78.269 ms stddev 48.148, 0 failed
progress: 540.0 s, 98.8 tps, lat 80.932 ms stddev 53.258, 0 failed
progress: 546.0 s, 99.5 tps, lat 80.046 ms stddev 56.527, 0 failed
progress: 552.0 s, 100.0 tps, lat 79.962 ms stddev 52.490, 0 failed
progress: 558.0 s, 97.2 tps, lat 82.153 ms stddev 56.429, 0 failed
progress: 564.0 s, 99.2 tps, lat 80.847 ms stddev 62.702, 0 failed
progress: 570.0 s, 100.5 tps, lat 79.354 ms stddev 50.785, 0 failed
progress: 576.0 s, 113.8 tps, lat 70.488 ms stddev 54.192, 0 failed
progress: 582.0 s, 85.7 tps, lat 93.229 ms stddev 66.400, 0 failed
progress: 588.0 s, 92.2 tps, lat 86.818 ms stddev 61.173, 0 failed
progress: 594.0 s, 104.5 tps, lat 76.478 ms stddev 54.135, 0 failed
progress: 600.0 s, 104.8 tps, lat 76.316 ms stddev 49.926, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
maximum number of tries: 1
duration: 600 s
number of transactions actually processed: 103568
number of failed transactions: 0 (0.000%)
latency average = 46.312 ms
latency stddev = 50.820 ms
initial connection time = 36.975 ms
tps = 172.596667 (without initial connection time)
```
5. Просмотрел объем сгенерированных файлов от зафиксированной позиции Wal.
```
postgres=# SELECT pg_size_pretty(pg_wal_lsn_diff((SELECT pg_current_wal_lsn()), '0/2227B008'));
 pg_size_pretty
----------------
 289 MB
(1 row)
```
6. Так как контрольные точки записывались каждые 30 секунд - а тест длился 10 минут(600 сек) - логично предположить что контрольных точек было 20.
```
289/20 = 14 MB
```
Следовательно - одна контрольная точка в среднем весила 14 мб.

7. Проверил логи - убедился, что контрольные точки выполнялись по расписанию
```
sudo tail /var/log/postgresql/postgresql-15-main.log
2025-05-10 20:30:48.189 UTC [819] LOG:  checkpoint starting: time
2025-05-10 20:31:15.185 UTC [819] LOG:  checkpoint complete: wrote 1451 buffers (8.9%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.762 s, sync=0.102 s, total=26.996 s; sync files=15, longest=0.021 s, average=0.007 s; distance=12553 kB, estimate=14659 kB
2025-05-10 20:31:18.187 UTC [819] LOG:  checkpoint starting: time
2025-05-10 20:31:45.174 UTC [819] LOG:  checkpoint complete: wrote 1232 buffers (7.5%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.799 s, sync=0.065 s, total=26.988 s; sync files=9, longest=0.021 s, average=0.008 s; distance=12400 kB, estimate=14433 kB
2025-05-10 20:31:48.180 UTC [819] LOG:  checkpoint starting: time
2025-05-10 20:32:15.175 UTC [819] LOG:  checkpoint complete: wrote 1202 buffers (7.3%); 0 WAL file(s) added, 0 removed, 0 recycled; write=26.797 s, sync=0.093 s, total=26.996 s; sync files=11, longest=0.019 s, average=0.009 s; distance=12624 kB, estimate=14252 kB
2025-05-10 20:32:18.180 UTC [819] LOG:  checkpoint starting: time
2025-05-10 20:32:45.113 UTC [819] LOG:  checkpoint complete: wrote 1240 buffers (7.6%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.719 s, sync=0.055 s, total=26.934 s; sync files=9, longest=0.018 s, average=0.007 s; distance=12472 kB, estimate=14074 kB
2025-05-10 20:33:48.179 UTC [819] LOG:  checkpoint starting: time
2025-05-10 20:34:15.190 UTC [819] LOG:  checkpoint complete: wrote 1125 buffers (6.9%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.841 s, sync=0.079 s, total=27.011 s; sync files=15, longest=0.018 s, average=0.006 s; distance=10659 kB, estimate=13733 kB
```
Как видно - все контрольные точки запускались по расписанию.

**Синхронный/Ассинхроный режим.**
	Основное отличие синхронного и ассинхронного режима записи заключается в том, что в синхронном режиме результат записи возвращается только после физической записи данных на диск, тогда как в ассинхронном режиме - ожидание физической записи не производится. За счет указанного отличия - ассинхронная запись должна быть существенно быстрее чем синхронная, но синхронная запись надежней за счет точного подтверждения результата.
	Чтобы узнать в каком режиме мы сейчас находимся - выполним команду - как видно мы находимся в синхронном режиме.
	```
	SHOW synchronous_commit;
	 synchronous_commit
	--------------------
	 on
	(1 row)
	```9. Проверим производительность базы в данном режиме:
```
postgres@ubuntu-vm:~$ pgbench -c 10 -P 10 -T 60
pgbench (15.12 (Ubuntu 15.12-1.pgdg24.04+1))
starting vacuum...end.
progress: 10.0 s, 345.6 tps, lat 28.718 ms stddev 22.700, 0 failed
progress: 20.0 s, 339.4 tps, lat 29.455 ms stddev 22.507, 0 failed
progress: 30.0 s, 334.0 tps, lat 29.871 ms stddev 23.148, 0 failed
progress: 40.0 s, 333.7 tps, lat 29.972 ms stddev 24.588, 0 failed
progress: 50.0 s, 335.5 tps, lat 29.833 ms stddev 23.628, 0 failed
progress: 60.0 s, 338.9 tps, lat 29.497 ms stddev 24.017, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 10
number of threads: 1
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 20280
number of failed transactions: 0 (0.000%)
latency average = 29.557 ms
latency stddev = 23.440 ms
initial connection time = 49.172 ms
tps = 338.071350 (without initial connection time)
```
10. Переключим режим на ассинхронный - выполнив команду.
```
ALTER SYSTEM SET synchronous_commit = 'off';
ALTER SYSTEM
```
11. Повторно проверим производительность базы.
```
postgres@ubuntu-vm:~$ pgbench -c 10 -P 10 -T 60
pgbench (15.12 (Ubuntu 15.12-1.pgdg24.04+1))
starting vacuum...end.
progress: 10.0 s, 1016.9 tps, lat 9.774 ms stddev 6.153, 0 failed
progress: 20.0 s, 958.1 tps, lat 10.423 ms stddev 6.423, 0 failed
progress: 30.0 s, 989.3 tps, lat 10.095 ms stddev 6.317, 0 failed
progress: 40.0 s, 941.7 tps, lat 10.611 ms stddev 6.689, 0 failed
progress: 50.0 s, 969.2 tps, lat 10.305 ms stddev 6.375, 0 failed
progress: 60.0 s, 965.6 tps, lat 10.342 ms stddev 6.516, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 10
number of threads: 1
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 58418
number of failed transactions: 0 (0.000%)
latency average = 10.254 ms
latency stddev = 6.419 ms
initial connection time = 40.578 ms
tps = 973.724295 (without initial connection time)
```
Как видно - наши догадки подтвердились - ассинхронный режим существенно быстрее синхронного, но синхронный - надежней.
**Контрольные суммы.**
12. Для начала создадим и запустим новый кластер у которого будет включена проверка контрольной суммы.
```
sudo pg_createcluster --start-conf manual 15 checksumdb
sudo pg_ctlcluster 15 checksumdb start
```
13. Просматриваем список кластеров - смотрим на каком порту работает наш кластер.
```
pg_lsclusters
Ver Cluster      Port Status Owner    Data directory                      Log file
15  checksumdb   5434 online postgres /var/lib/postgresql/15/checksumdb   /var/log/postgresql/postgresql-15-checksumdb.log
15  main         5433 online postgres /mnt/data/15/main                   /var/log/postgresql/postgresql-15-main.log
```
14. Подключаемся к новому кластеру, создаем БД, заполняем ее данными и узнаем физический пусть к таблице.
```
psql -p 5434
psql (17.4 (Ubuntu 17.4-1.pgdg24.04+2), server 15.12 (Ubuntu 15.12-1.pgdg24.04+1))
Type "help" for help.

postgres=# create table test_table (testfield text);
CREATE TABLE
                                      
postgres=# insert into test_table values ('test0'), ('test1'), ('test2');
INSERT 0 3
postgres=# select pg_relation_filepath('test_table');
 pg_relation_filepath
----------------------
 base/5/16388
(1 row)
```
15. Остановил кластер
```
sudo pg_ctlcluster 15 checksumdb stop
```
16. Открыл файл базы - внес в него изменения (сохранив размер):
```
sudo nano /var/lib/postgresql/15/checksumdb/base/5/16388

(выполнил изменения одного символа)
test2 -> tes12
```
17. Запстил кластер и попробовал получить данные - получил ошибку:
```
ERROR:  invalid page in block 0 of relation base/5/16388 
```
Ошибка сообщает о том, что данные были повреждены - без контрольной суммы - мы бы об этом даже не узнали.

