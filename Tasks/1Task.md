1. Создаем VM с ubuntu, устанавливаем на нее Postgress.
```sudo apt install postgresql-15```
2. Подключаемся к VM с двух SSH.
Открываем psql.
```sudo -u postgres psql```
3. Отключаем актокомит.
```\set AUTOCOMMIT off```
4. Создаем и заполняем таблицу:
```
create table persons(id serial, first_name text, second_name text);
insert into persons(first_name, second_name) values('ivan', 'ivanov');
insert into persons(first_name, second_name) values('petr', 'petrov');
commit;
```
5. Смотрим уровень изоляции.
```
show transaction isolation level;
transaction_isolation

-----------------------
 read committed
(1 row)
```
6. В обоих сессиям начинаем транзакцию:
```
postgres=# begin transaction;
BEGIN
```
7. В первой сессии добавляем запись:
```
postgres=*# insert into persons(first_name, second_name) values('sergey', 'sergeev');
INSERT 0 1
```
8. Во вторую - делаем выборку из таблицы persons.
```
postgres=*# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
(2 rows)
```
Как видно - добавленной записи тут не видно - так как транзакция не была завершена.
9. Завершаем транзакцию в первой сессии.
```
postgres=*# commit;
COMMIT
```
10. Повторно делаем выборку во второй сессии.
```
select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 rows)
```
Теперь новая запись видна, так как транзакция бала завершена.
11. Завершаем транзакцию во второй сесии и повторно делаем запрос на получение записей.
```
postgres=*# commit;
COMMIT
postgres=# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 rows)
```
Мы по прежнему видим все три записи, так как наша транзакция не изменяла их.1. Создаем VM с ubuntu, устанавливаем на нее Postgress.
```sudo apt install postgresql-15```
12. Подключаемся к VM с двух SSH.
Открываем psql.
```sudo -u postgres psql```
13. Отключаем актокомит.
```\set AUTOCOMMIT off```
14. Создаем и заполняем таблицу:
```
create table persons(id serial, first_name text, second_name text);
insert into persons(first_name, second_name) values('ivan', 'ivanov');
insert into persons(first_name, second_name) values('petr', 'petrov');
commit;
```
15. Смотрим уровень изоляции.
``` 
show transaction isolation level;
 transaction_isolation
-----------------------
 read committed
(1 row)
```
17. В обоих сессиям начинаем транзакцию:
```
postgres=# begin transaction;
BEGIN
```
18. В первой сессии добавляем запись:
```
postgres=*# insert into persons(first_name, second_name) values('sergey', 'sergeev');
INSERT 0 1
```
19. Во вторую - делаем выборку из таблицы persons.
```
postgres=*# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
(2 rows)
```
Как видно - добавленной записи тут не видно - так как транзакция не была завершена.
20. Завершаем транзакцию в первой сессии.
```
postgres=*# commit;
COMMIT
```
21. Повторно делаем выборку во второй сессии.
```
select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 rows)
```
Теперь новая запись видна, так как транзакция бала завершена.
22. Завершаем транзакцию во второй сесии и повторно делаем запрос на получение записей.
```
postgres=*# commit;
COMMIT
postgres=# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 rows)
```
Мы по прежнему видим все три записи, так как наша транзакция не изменяла их.