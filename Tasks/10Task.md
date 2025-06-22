**Анализ**
1. Первым делом - проанализируем нашу демо базу (таблицу booking).
Как видно - таблица состоит из трех колонок 
```
 book_ref |       book_date        | total_amount
----------+------------------------+--------------
 000004   | 2016-08-13 12:40:00+00 |     55800.00
 00000F   | 2017-07-05 00:12:00+00 |    265700.00
 000010   | 2017-01-08 16:45:00+00 |     50900.00
 000012   | 2017-07-14 06:02:00+00 |     37900.00
 000026   | 2016-08-30 08:08:00+00 |     95600.00
 00002D   | 2017-05-20 15:45:00+00 |    114700.00
 000034   | 2016-08-08 02:46:00+00 |     49100.00
 00003F   | 2016-12-12 12:02:00+00 |    109800.00
 000048   | 2016-09-16 22:57:00+00 |     92400.00
 00004A   | 2016-10-13 18:57:00+00 |     29000.00
 000050   | 2016-09-17 21:01:00+00 |     36200.00
 000055   | 2017-03-08 19:18:00+00 |     50800.00
 000061   | 2016-11-11 18:28:00+00 |     35600.00
 000067   | 2016-08-11 18:36:00+00 |    102100.00
 000068   | 2017-08-15 11:27:00+00 |     18100.00
 00006A   | 2016-11-05 02:02:00+00 |    106100.00
 00006B   | 2016-11-29 04:59:00+00 |    382000.00
 00007A   | 2016-10-18 17:55:00+00 |      8200.00
 00007C   | 2016-09-26 20:00:00+00 |     22600.00
 00007D   | 2016-12-31 16:49:00+00 |    103600.00
 00008F   | 2017-03-16 17:23:00+00 |     75600.00
 000090   | 2016-10-23 13:26:00+00 |     22600.00
```
Секционировать значения можно по любой колонке, но для начала - попробуем сгрупировать данные, чтобы понять - в какой из групп будет смысл.
Предположим, что со стороны бизнеса часто есть смысл получать информацию о полетах в рамка определенного месяца - из-за большого объема данных - сделать это быстро в рамках исходной структуры довольно трудно. Помимо прочего - старые данные скорее всего со временем нужно будет удалять из системы, выполнив перенос в другое место.
2. Сгрупируем данные по годам:
```
demo=# select extract(year from book_date), count(book_date) from bookings group by extract(year from book_date);
 extract |  count
---------+---------
    2016 |  852935
    2017 | 1258175
(2 rows)
```
Видно, что по годам разделение не слишком равномерно.

3. Сгрупируем данные по месяцам - тут уже существенно лучше.
```
demo=# select extract(year from book_date) as year, extract(month from book_date) as mounth, count(book_date) from bookings group by (extract(year from book_date), extract(month from book_date));
 year | mounth | count
------+--------+--------
 2016 |      7 |  11394
 2016 |      8 | 168470
 2016 |      9 | 165419
 2016 |     10 | 170925
 2016 |     11 | 165437
 2016 |     12 | 171290
 2017 |      1 | 171206
 2017 |      2 | 154598
 2017 |      3 | 171260
 2017 |      4 | 165485
 2017 |      5 | 170952
 2017 |      6 | 165213
 2017 |      7 | 171671
 2017 |      8 |  87790
(14 rows)
```
4. Для оценки производительности - предлагаю сможелировать ситуацию, при которой нам нужно получить общую сумму бронирования за каждый месяц.
```
demo=# EXPLAIN (ANALYZE, BUFFERS) select extract(year from book_date) as year, extract(month from book_date) as mounth, count(book_date), sum(total_amount) from bookings group by (extract(year from book_date), extract(month from book_date));
                                                                     QUERY PLAN
----------------------------------------------------------------------------------------------------------------------------------------------------
 Finalize GroupAggregate  (cost=192665.59..333920.57 rows=447859 width=104) (actual time=2429.097..2863.788 rows=14 loops=1)
   Group Key: (EXTRACT(year FROM book_date)), (EXTRACT(month FROM book_date))
   Buffers: shared hit=817 read=12742, temp read=21170 written=21218
   ->  Gather Merge  (cost=192665.59..314886.56 rows=895718 width=104) (actual time=2367.734..2863.459 rows=42 loops=1)
         Workers Planned: 2
         Workers Launched: 2
         Buffers: shared hit=817 read=12742, temp read=21170 written=21218
         ->  Partial GroupAggregate  (cost=191665.56..210498.46 rows=447859 width=104) (actual time=1867.602..2267.695 rows=14 loops=3)
               Group Key: (EXTRACT(year FROM book_date)), (EXTRACT(month FROM book_date))
               Buffers: shared hit=817 read=12742, temp read=21170 written=21218
               ->  Sort  (cost=191665.56..193864.64 rows=879629 width=78) (actual time=1865.823..2051.385 rows=703703 loops=3)
                     Sort Key: (EXTRACT(year FROM book_date)), (EXTRACT(month FROM book_date))
                     Sort Method: external merge  Disk: 28744kB
                     Buffers: shared hit=817 read=12742, temp read=21170 written=21218
                     Worker 0:  Sort Method: external merge  Disk: 29816kB
                     Worker 1:  Sort Method: external merge  Disk: 26248kB
                     ->  Parallel Seq Scan on bookings  (cost=0.00..26641.44 rows=879629 width=78) (actual time=8.196..726.194 rows=703703 loops=3)
                           Buffers: shared hit=705 read=12742
 Planning Time: 0.079 ms
 JIT:
   Functions: 24
   Options: Inlining false, Optimization false, Expressions true, Deforming true
   Timing: Generation 2.713 ms, Inlining 0.000 ms, Optimization 0.857 ms, Emission 23.659 ms, Total 27.229 ms
 Execution Time: 2875.644 ms
(24 rows)
```
В связи с огромным объемом выборки - сейчас запрос выполняется почти 3 секунды.

**Секционирование**.
5. Для начала - посмотрим структуру существующей таблицы.
```
demo=# \d+ bookings
                                                       Table "bookings.bookings"
    Column    |           Type           | Collation | Nullable | Default | Storage  | Compression | Stats target |    Description
--------------+--------------------------+-----------+----------+---------+----------+-------------+--------------+--------------------
 book_ref     | character(6)             |           | not null |         | extended |             |              | Booking number
 book_date    | timestamp with time zone |           | not null |         | plain    |             |              | Booking date
 total_amount | numeric(10,2)            |           | not null |         | main     |             |              | Total booking cost
Indexes:
    "bookings_pkey" PRIMARY KEY, btree (book_ref)
Referenced by:
    TABLE "tickets" CONSTRAINT "tickets_book_ref_fkey" FOREIGN KEY (book_ref) REFERENCES bookings(book_ref)
Access method: heap
```
6. Создадим таблицу - укажем в качестве ключа разделения - book_date. Так же включим данное поле в primary key.
```
CREATE TABLE bookings_sectioned (  
    book_ref character(6)not null,  
    book_date timestamp with time zone not null,  
    total_amount numeric(10,2) not null,  
    primary key (book_ref, book_date)  
) partition by range (book_date);  
CREATE TABLE
```
7. Теперь нам надо создать партиции - для начала подготовим запрос, который выдаст даты начала и конца по каждому из месяцев в таблице.
```
demo=# select date_trunc('month', book_date) as startPartition, date_trunc('month', book_date) + INTERVAL '1 month' as endPartition from bookings group by date_trunc('month', book_date);
     startpartition     |      endpartition
------------------------+------------------------
 2017-07-01 00:00:00+00 | 2017-08-01 00:00:00+00
 2017-06-01 00:00:00+00 | 2017-07-01 00:00:00+00
 2016-10-01 00:00:00+00 | 2016-11-01 00:00:00+00
 2017-02-01 00:00:00+00 | 2017-03-01 00:00:00+00
 2017-01-01 00:00:00+00 | 2017-02-01 00:00:00+00
 2017-03-01 00:00:00+00 | 2017-04-01 00:00:00+00
 2017-08-01 00:00:00+00 | 2017-09-01 00:00:00+00
 2016-11-01 00:00:00+00 | 2016-12-01 00:00:00+00
 2016-09-01 00:00:00+00 | 2016-10-01 00:00:00+00
 2016-07-01 00:00:00+00 | 2016-08-01 00:00:00+00
 2016-08-01 00:00:00+00 | 2016-09-01 00:00:00+00
 2017-05-01 00:00:00+00 | 2017-06-01 00:00:00+00
 2017-04-01 00:00:00+00 | 2017-05-01 00:00:00+00
 2016-12-01 00:00:00+00 | 2017-01-01 00:00:00+00
(14 rows)
```
8. Соберем простой скрипт - для создания партиций.
```
DO $$
DECLARE
    p record;
	partition_name text;
BEGIN
    FOR p IN 
		select date_trunc('month', book_date) as startPartition, date_trunc('month', book_date) + INTERVAL '1 month' as endPartition from bookings group by date_trunc('month', book_date)
    LOOP
		partition_name := to_char(p.startPartition, 'YYYY"m"MM');
        EXECUTE format(
            'CREATE TABLE bookings_sectioned_%s PARTITION OF bookings_sectioned '
            'FOR VALUES FROM (%L) TO (%L)',
            partition_name, 
            p.startPartition, 
            p.endPartition
        );
    END LOOP;
END;
$$;
```
9. Выполним его и проверим результат.
```
demo=# SELECT * FROM pg_partition_tree('bookings_sectioned');
           relid            |    parentrelid     | isleaf | level
----------------------------+--------------------+--------+-------
 bookings_sectioned         |                    | f      |     0
 bookings_sectioned_2017m07 | bookings_sectioned | t      |     1
 bookings_sectioned_2017m06 | bookings_sectioned | t      |     1
 bookings_sectioned_2016m10 | bookings_sectioned | t      |     1
 bookings_sectioned_2017m02 | bookings_sectioned | t      |     1
 bookings_sectioned_2017m01 | bookings_sectioned | t      |     1
 bookings_sectioned_2017m03 | bookings_sectioned | t      |     1
 bookings_sectioned_2017m08 | bookings_sectioned | t      |     1
 bookings_sectioned_2016m11 | bookings_sectioned | t      |     1
 bookings_sectioned_2016m09 | bookings_sectioned | t      |     1
 bookings_sectioned_2016m07 | bookings_sectioned | t      |     1
 bookings_sectioned_2016m08 | bookings_sectioned | t      |     1
 bookings_sectioned_2017m05 | bookings_sectioned | t      |     1
 bookings_sectioned_2017m04 | bookings_sectioned | t      |     1
 bookings_sectioned_2016m12 | bookings_sectioned | t      |     1
(15 rows)
```
10. Скопируем данные в новую таблицу.
```
insert into bookings_sectioned select * from bookings;
INSERT 0 2111110
```
11. Убедимся, что мы скопировали все записи - сверим количество.
```
demo=# select count(*) from bookings;
  count
---------
 2111110
(1 row)

demo=# select count(*) from bookings_sectioned;
  count
---------
 2111110
(1 row)
```
12. Проверим размеры партиций - как видим, разделение получислось равномерным.
```
SELECT
    child.relname AS partition_name,
    regexp_replace(pg_get_expr(child.relpartbound, child.oid),
                   'FOR VALUES FROM \(''(.*)''\) TO \(''(.*)''\)',
                   'От \1 до \2') AS readable_range,
    pg_size_pretty(pg_total_relation_size(child.oid)) AS size
FROM pg_inherits
JOIN pg_class parent ON pg_inherits.inhparent = parent.oid
JOIN pg_class child ON pg_inherits.inhrelid = child.oid
WHERE parent.relname = 'bookings_sectioned' order by partition_name;
       partition_name       |                   readable_range                    |  size
----------------------------+-----------------------------------------------------+---------
 bookings_sectioned_2016m07 | От 2016-07-01 00:00:00+00 до 2016-08-01 00:00:00+00 | 984 kB
 bookings_sectioned_2016m08 | От 2016-08-01 00:00:00+00 до 2016-09-01 00:00:00+00 | 14 MB
 bookings_sectioned_2016m09 | От 2016-09-01 00:00:00+00 до 2016-10-01 00:00:00+00 | 13 MB
 bookings_sectioned_2016m10 | От 2016-10-01 00:00:00+00 до 2016-11-01 00:00:00+00 | 14 MB
 bookings_sectioned_2016m11 | От 2016-11-01 00:00:00+00 до 2016-12-01 00:00:00+00 | 13 MB
 bookings_sectioned_2016m12 | От 2016-12-01 00:00:00+00 до 2017-01-01 00:00:00+00 | 14 MB
 bookings_sectioned_2017m01 | От 2017-01-01 00:00:00+00 до 2017-02-01 00:00:00+00 | 14 MB
 bookings_sectioned_2017m02 | От 2017-02-01 00:00:00+00 до 2017-03-01 00:00:00+00 | 12 MB
 bookings_sectioned_2017m03 | От 2017-03-01 00:00:00+00 до 2017-04-01 00:00:00+00 | 14 MB
 bookings_sectioned_2017m04 | От 2017-04-01 00:00:00+00 до 2017-05-01 00:00:00+00 | 13 MB
 bookings_sectioned_2017m05 | От 2017-05-01 00:00:00+00 до 2017-06-01 00:00:00+00 | 14 MB
 bookings_sectioned_2017m06 | От 2017-06-01 00:00:00+00 до 2017-07-01 00:00:00+00 | 13 MB
 bookings_sectioned_2017m07 | От 2017-07-01 00:00:00+00 до 2017-08-01 00:00:00+00 | 14 MB
 bookings_sectioned_2017m08 | От 2017-08-01 00:00:00+00 до 2017-09-01 00:00:00+00 | 7224 kB
(14 rows)
```
13. Нам осталось только проверить скорость выполнения запроса по секционированной таблице.
```
demo=# EXPLAIN (ANALYZE, BUFFERS) select extract(year from book_date) as year, extract(month from book_date) as mounth, count(book_date), sum(total_amount) from bookings_sectioned group by (extract(year from book_date), extract(month from book_date));
                                                                                           QUERY PLAN
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 Finalize GroupAggregate  (cost=46287.04..46342.21 rows=200 width=104) (actual time=1264.765..1274.111 rows=14 loops=1)
   Group Key: (EXTRACT(year FROM bookings_sectioned.book_date)), (EXTRACT(month FROM bookings_sectioned.book_date))
   Buffers: shared hit=13468
   ->  Gather Merge  (cost=46287.04..46333.71 rows=400 width=104) (actual time=1264.755..1274.088 rows=17 loops=1)
         Workers Planned: 2
         Workers Launched: 2
         Buffers: shared hit=13468
         ->  Sort  (cost=45287.01..45287.51 rows=200 width=104) (actual time=1223.080..1223.086 rows=6 loops=3)
               Sort Key: (EXTRACT(year FROM bookings_sectioned.book_date)), (EXTRACT(month FROM bookings_sectioned.book_date))
               Sort Method: quicksort  Memory: 26kB
               Buffers: shared hit=13468
               Worker 0:  Sort Method: quicksort  Memory: 25kB
               Worker 1:  Sort Method: quicksort  Memory: 25kB
               ->  Partial HashAggregate  (cost=45275.87..45279.37 rows=200 width=104) (actual time=1223.037..1223.045 rows=6 loops=3)
                     Group Key: (EXTRACT(year FROM bookings_sectioned.book_date)), (EXTRACT(month FROM bookings_sectioned.book_date))
                     Batches: 1  Memory Usage: 40kB
                     Buffers: shared hit=13454
                     Worker 0:  Batches: 1  Memory Usage: 40kB
                     Worker 1:  Batches: 1  Memory Usage: 40kB
                     ->  Parallel Append  (cost=0.00..36479.58 rows=879629 width=78) (actual time=0.629..826.642 rows=703703 loops=3)
                           Buffers: shared hit=13454
                           ->  Parallel Seq Scan on bookings_sectioned_2017m07 bookings_sectioned_13  (cost=0.00..2608.74 rows=100983 width=78) (actual time=0.026..300.767 rows=171671 loops=1)
                                 Buffers: shared hit=1094
                           ->  Parallel Seq Scan on bookings_sectioned_2016m12 bookings_sectioned_6  (cost=0.00..2603.38 rows=100759 width=78) (actual time=0.023..113.169 rows=171290 loops=1)
                                 Buffers: shared hit=1092
                           ->  Parallel Seq Scan on bookings_sectioned_2017m03 bookings_sectioned_9  (cost=0.00..2602.12 rows=100741 width=78) (actual time=0.014..124.909 rows=171260 loops=1)
                                 Buffers: shared hit=1091
                           ->  Parallel Seq Scan on bookings_sectioned_2017m01 bookings_sectioned_7  (cost=0.00..2601.64 rows=100709 width=78) (actual time=0.013..238.861 rows=171206 loops=1)
                                 Buffers: shared hit=1091
                           ->  Parallel Seq Scan on bookings_sectioned_2017m05 bookings_sectioned_11  (cost=0.00..2597.40 rows=100560 width=78) (actual time=0.005..189.288 rows=170952 loops=1)
                                 Buffers: shared hit=1089
                           ->  Parallel Seq Scan on bookings_sectioned_2016m10 bookings_sectioned_4  (cost=0.00..2597.16 rows=100544 width=78) (actual time=0.006..134.361 rows=170925 loops=1)
                                 Buffers: shared hit=1089
                           ->  Parallel Seq Scan on bookings_sectioned_2016m08 bookings_sectioned_2  (cost=0.00..2560.50 rows=99100 width=78) (actual time=0.005..127.350 rows=56157 loops=3)
                                 Buffers: shared hit=1074
                           ->  Parallel Seq Scan on bookings_sectioned_2017m04 bookings_sectioned_10  (cost=0.00..2515.16 rows=97344 width=78) (actual time=0.006..37.693 rows=82742 loops=2)
                                 Buffers: shared hit=1055
                           ->  Parallel Seq Scan on bookings_sectioned_2016m11 bookings_sectioned_5  (cost=0.00..2513.74 rows=97316 width=78) (actual time=0.005..93.890 rows=165437 loops=1)
                                 Buffers: shared hit=1054
                           ->  Parallel Seq Scan on bookings_sectioned_2016m09 bookings_sectioned_3  (cost=0.00..2513.58 rows=97305 width=78) (actual time=0.008..131.832 rows=165419 loops=1)
                                 Buffers: shared hit=1054
                           ->  Parallel Seq Scan on bookings_sectioned_2017m06 bookings_sectioned_12  (cost=0.00..2510.76 rows=97184 width=78) (actual time=0.007..169.817 rows=165213 loops=1)
                                 Buffers: shared hit=1053
                           ->  Parallel Seq Scan on bookings_sectioned_2017m02 bookings_sectioned_8  (cost=0.00..2349.10 rows=90940 width=78) (actual time=0.006..185.541 rows=154598 loops=1)
                                 Buffers: shared hit=985
                           ->  Parallel Seq Scan on bookings_sectioned_2017m08 bookings_sectioned_14  (cost=0.00..1334.62 rows=51641 width=78) (actual time=0.006..77.417 rows=87790 loops=1)
                                 Buffers: shared hit=560
                           ->  Parallel Seq Scan on bookings_sectioned_2016m07 bookings_sectioned_1  (cost=0.00..173.53 rows=6702 width=78) (actual time=1.834..7.413 rows=11394 loops=1)
                                 Buffers: shared hit=73
 Planning Time: 0.510 ms
 Execution Time: 1274.181 ms
(51 rows)
```
Как видно, скорость обработки запроса существенно выросла, но это актуально только для действительно больших таблиц, где ситуации с получениям данных из разных партиций будут довольно редки.
