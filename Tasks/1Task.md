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
Мы по прежнему видим все три записи, так как наша транзакция не изменяла их.

12.  Начинаем новые транзакции c уровнем транзакции repeatable read.
```
postgres=# begin transaction isolation level repeatable read;
BEGIN
```
13. В первой сессии делаем insert:
```
insert into persons(first_name, second_name) values('sveta', 'svetova');
INSERT 0 1
```
14. Получаем данные таблицы persons во второй сессии:
```
select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 rows)
```
Так как транзакция, в которой были созданы данные еще не завершена - созданные данные не отображаются в овтете.
15. Завершаем транзакцию в первой сессии.
```
postgres=*# commit;
COMMIT
```
16. Делаем выборку во второй сессии:
```
postgres=*# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 rows)
```
Новую запись мы не видим, так как уровень изоляции repeatable read блокирует состояние до завершения транзакции.
17. Завершаем транзакцию во второй сессии.
```
postgres=*# commit;
COMMIT
```
18. Повторим запрос.
```
postgres=# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
  4 | sveta      | svetova
(4 rows)
```
Как только мы завершили нашу транзакцию - новый запрос выполняется с уровнем изолации READ COMMITTED, что позволяет нам видеть данные, добавленные вне нашей транзакции.