1. Для работы с индексами - нам потребуктся какой-то набор данных - в качестве примера был выбран просто список колледжей США - [https://github.com/nytimes/covid-19-data/blob/master/colleges/colleges.csv](https://github.com/nytimes/covid-19-data/blob/master/colleges/colleges.csv)
1. Для примера из данной таблицы мы возьмем только данные о коледжах - штат, регион, город и название города - создадим для хранения этого всего таблицу.
```
postgres=# create table collages(id int, state varchar(32), county varchar(32), city varchar(32), name varchar(255));
```
2. Закинем таблицу с нужными колонками на нашу систему и возьмем из нее данные для заполнения.
```
\copy collages FROM '/tmp/colleges.csv' DELIMITER ',' CSV
```
3. Проверим корректность данных.
```
postgres=# select * from collages;
    id    |          state           |            county            |          city           |                                     name
----------+--------------------------+------------------------------+-------------------------+-------------------------------------------------------------------------------
   100654 | Alabama                  | Madison                      | Huntsville              | Alabama A&M University
   100724 | Alabama                  | Montgomery                   | Montgomery              | Alabama State University
   100812 | Alabama                  | Limestone                    | Athens                  | Athens State University
   100858 | Alabama                  | Lee                          | Auburn                  | Auburn University
   100830 | Alabama                  | Montgomery                   | Montgomery              | Auburn University at Montgomery
   102429 | Alabama                  | Walker                       | Jasper                  | Bevill State Community College
   100937 | Alabama                  | Jefferson                    | Birmingham              | Birmingham-Southern College
   101514 | Alabama                  | Limestone                    | Tanner                  | Calhoun Community College
   100760 | Alabama                  | Tallapoosa                   | Alexander City          | Central Alabama Community College
   101143 | Alabama                  | Coffee                       | Enterprise              | Enterprise State Community College
   101240 | Alabama                  | Etowah                       | Gadsden                 | Gadsden State Community College
   101435 | Alabama                  | Montgomery                   | Montgomery              | Huntingdon College
   101480 | Alabama                  | Calhoun                      | Jacksonville            | Jacksonville State University
   101602 | Alabama                  | Covington                    | Andalusia               | Lurleen B. Wallace Community College
   101648 | Alabama                  | Perry                        | Marion                  | Marion Military Institute
   101675 | Alabama                  | Jefferson                    | Fairfield               | Miles College
   102049 | Alabama                  | Jefferson                    | Birmingham              | Samford University
   102076 | Alabama                  | Marshall                     | Boaz                    | Snead State Community College
   251260 | Alabama                  | Randolph                     | Wadley                  | Southern Union State Community College
   102234 | Alabama                  | Mobile                       | Mobile                  | Spring Hill College
   102368 | Alabama                  | Pike                         | Troy                    | Troy University
 10236803 | Alabama                  | Houston                      | Dothan                  | Troy University Dothan
 10236802 | Alabama                  | Montgomery                   | Montgomery              | Troy University Montgomery
 10236801 | Alabama                  | Russell                      | Phenix                  | Troy University Phenix City
   102377 | Alabama                  | Macon                        | Tuskeegee               | Tuskegee University
   100751 | Alabama                  | Tuscaloosa                   | Tuscaloosa              | University of Alabama
   100663 | Alabama                  | Jefferson                    | Birmingham              | University of Alabama at Birmingham
   100706 | Alabama                  | Madison                      | Huntsville              | University of Alabama in Huntsville
   101709 | Alabama                  | Shelby                       | Montevallo              | University of Montevallo
   101879 | Alabama                  | Lauderdale                   | Florence                | University of North Alabama
   102094 | Alabama                  | Mobile                       | Mobile                  | University of South Alabama
   101587 | Alabama                  | Sumter                       | Livingston              | University of West Alabama
   434584 | Alaska                   | North Slope Borough          | Utqiagvik               | Ilisagvik College
   102553 | Alaska                   | Anchorage Municipality       | Anchorage               | University of Alaska Anchorage
   102614 | Alaska                   | Fairbanks North Star Borough | Fairbanks               | University of Alaska Fairbanks
   102632 | Alaska                   | Juneau City and Borough      | Juneau                  | University of Alaska Southeast
   240736 | American Samoa           | Maoputasi                    | Pago Pago               | American Samoa Community College
   104151 | Arizona                  | Maricopa                     | Tempe                   | Arizona State University
   104160 | Arizona                  | Yuma                         | Yuma                    | Arizona Western College
   105297 | Arizona                  | Apache                       | Tsaile                  | Dine College
   104586 | Arizona                  | Yavapai                      | Prescott                | Embry-Riddle Aeronautical University at Prescott
   104717 | Arizona                  | Maricopa                     | Phoenix                 | Grand Canyon University
   423643 | Arizona                  | Maricopa                     | Glendale                | Midwestern University at Glendale
   105206 | Arizona                  | Mohave                       | Kingman                 | Mohave Community College
   105330 | Arizona                  | Coconino                     | Flagstaff               | Northern Arizona University
   104179 | Arizona                  | Pima                         | Tucson                  | University of Arizona
   107327 | Arkansas                 | Mississippi                  | Blytheville             | Arkansas Northeastern College
   106458 | Arkansas                 | Craighead                    | Jonesboro               | Arkansas State University
   440402 | Arkansas                 | Jackson                      | Newport                 | Arkansas State University-Newport
   106467 | Arkansas                 | Pope                         | Russellville            | Arkansas Tech University
   106625 | Arkansas                 | Randolph                     | Pocahontas              | Black River Technical College
   106713 | Arkansas                 | Faulkner                     | Conway                  | Central Baptist College
   106795 | Arkansas                 | Sevier                       | De Queen                | Cossatot Community College of the University of Arkansas
   107044 | Arkansas                 | White                        | Searcy                  | Harding University
   107071 | Arkansas                 | Clark                        | Arkadelphia             | Henderson State University
   107080 | Arkansas                 | Faulkner                     | Conway                  | Hendrix College
   107141 | Arkansas                 | Benton                       | Siloam Springs          | John Brown University
   106342 | Arkansas                 | Independence                 | Batesville              | Lyon College
   106980 | Arkansas                 | Garland                      | Hot Springs             | National Park College
   107460 | Arkansas                 | Boone                        | Harrison                | North Arkansas College
   367459 | Arkansas                 | Benton                       | Bentonville             | NorthWest Arkansas Community College
   107512 | Arkansas                 | Clark                        | Arkadelphia             | Ouachita Baptist University
   107974 | Arkansas                 | Union                        | El Dorado               | South Arkansas Community College
   107983 | Arkansas                 | Columbia                     | Magnolia                | Southern Arkansas University
   106397 | Arkansas                 | Washington                   | Fayetteville            | University of Arkansas
   107743 | Arkansas                 | Polk                         | Mena                    | University of Arkansas Community College Rich Mountain
   107585 | Arkansas                 | Conway                       | Morrilton               | University of Arkansas Community College at Morrilton
 10772501 | Arkansas                 | Miller                       | Texarkana               | University of Arkansas Hope-Texarkana
   108092 | Arkansas                 | Sebastian                    | Fort Smith              | University of Arkansas at Fort Smith
   106245 | Arkansas                 | Pulaski                      | Little Rock             | University of Arkansas at Little Rock
   106485 | Arkansas                 | Drew                         | Monticello              | University of Arkansas at Monticello
   106412 | Arkansas                 | Jefferson                    | Pine Bluff              | University of Arkansas at Pine Bluff
   106263 | Arkansas                 | Pulaski                      | Little Rock             | University of Arkansas for Medical Sciences
   107664 | Arkansas                 | Pulaski                      | North Little Rock       | University of Arkansas-Pulaski Technical College
   106704 | Arkansas                 | Conway                       | Conway                  | University of Central Arkansas
   107558 | Arkansas                 | Johnson                      | Clarksville             | University of the Ozarks
   108232 | California               | San Francisco                | San Francisco           | Academy of Art University
   109350 | California               | Los Angeles                  | Lancaster               | Antelope Valley College
   109785 | California               | Los Angeles                  | Azusa                   | Azusa Pacific University
   109819 | California               | Kern                         | Bakersfield             | Bakersfield College
   110097 | California               | Los Angeles                  | La Mirada               | Biola University
   110361 | California               | Riverside                    | Riverside               | California Baptist University
   110404 | California               | Los Angeles                  | Pasadena                | California Institute of Technology
   110413 | California               | Ventura                      | Thousand Oaks           | California Lutheran University
   110422 | California               | San Luis Obispo              | San Luis Obispo         | California Polytechnic State University, San Luis Obispo
   110529 | California               | Los Angeles                  | Pomona                  | California State Polytechnic University, Pomona
   111188 | California               | Solano                       | Vallejo                 | California State University Maritime Academy
   110486 | California               | Kern                         | Bakersfield             | California State University, Bakersfield
   441937 | California               | Ventura                      | Camarillo               | California State University, Channel Islands
   110538 | California               | Butte                        | Chico                   | California State University, Chico
   110547 | California               | Los Angeles                  | Carson                  | California State University, Dominguez Hills
   110574 | California               | Alameda                      | Hayward                 | California State University, East Bay
   110565 | California               | Orange                       | Fullerton               | California State University, Fullerton
   110583 | California               | Los Angeles                  | Long Beach              | California State University, Long Beach
   110592 | California               | Los Angeles                  | Los Angeles             | California State University, Los Angeles
   409698 | California               | Monterey                     | Marina                  | California State University, Monterey Bay
   110608 | California               | Los Angeles                  | Los Angeles             | California State University, Northridge
   110617 | California               | Sacramento                   | Sacramento              | California State University, Sacramento
   110510 | California               | San Bernardino               | San Bernardino          | California State University, San Bernardino
   366711 | California               | San Diego                    | San Marcos              | California State University, San Marcos
--More--Cancel request sent
   110495 | California               | Stanislaus                   | Turlock                 | California State Univers
ity, Stanislaus
   111948 | California               | Orange                       | Orange                  | Chapman University
   112260 | California               | Los Angeles                  | Claremont               | Claremont McKenna Colleg
e
   112075 | California               | Orange                       | Irvine                  | Concordia University Irv
ine
   113236 | California               | Orange                       | Cypress                 | Cypress College
   113698 | California               | Marin                        | San Rafael              | Dominican University of
California
   113980 | California               | Los Angeles                  | Torrance                | El Camino Community Coll
ege
   114433 | California               | Plumas                       | Quincy                  | Feather River College
   114716 | California               | Santa Clara                  | Los Altos               | Foothill College
   114813 | California               | Fresno                       | Fresno                  | Fresno Pacific Universit
y
   110556 | California               | Fresno                       | Fresno                  | Fresno State University
   115409 | California               | Los Angeles                  | Claremont               | Harvey Mudd College
   115728 | California               | Alameda                      | Oakland                 | Holy Names University
   115755 | California               | Humboldt                     | Arcata                  | Humboldt State Universit
y
   117946 | California               | Los Angeles                  | Los Angeles             | Loyola Marymount Univers
ity
   118693 | California               | San Mateo                    | Atherton                | Menlo College
   118888 | California               | Alameda                      | Oakland                 | Mills College
   118912 | California               | San Diego                    | Oceanside               | MiraCosta College
   118976 | California               | Stanislaus                   | Modesto                 | Modesto Junior College
   119678 | California               | Monterey                     | Monterey                | Naval Postgraduate Schoo
l
```
4. Для начала - оценим выборку колледжей по штату California.
```
postgres=# EXPLAIN (ANALYZE, BUFFERS) Select * from collages where state = 'California';
                                               QUERY PLAN
--------------------------------------------------------------------------------------------------------
 Seq Scan on collages  (cost=0.00..92.68 rows=178 width=57) (actual time=0.013..0.313 rows=178 loops=1)
   Filter: ((state)::text = 'California'::text)
   Rows Removed by Filter: 3716
   Buffers: shared hit=44
 Planning:
   Buffers: shared hit=87
 Planning Time: 0.346 ms
 Execution Time: 0.345 ms
(8 rows)
```
5. Создадим индекс по данному полю.
```
postgres=# CREATE INDEX idx_collage_state ON collages(state);
CREATE INDEX
```
6. Проверим план запроса повторно.
```
postgres=# EXPLAIN (ANALYZE, BUFFERS) Select * from collages where state = 'California';
                                                          QUERY PLAN
------------------------------------------------------------------------------------------------------------------------------
 Bitmap Heap Scan on collages  (cost=5.66..51.88 rows=178 width=57) (actual time=0.096..0.114 rows=178 loops=1)
   Recheck Cond: ((state)::text = 'California'::text)
   Heap Blocks: exact=4
   Buffers: shared hit=4 read=2
   ->  Bitmap Index Scan on idx_collage_state  (cost=0.00..5.62 rows=178 width=0) (actual time=0.091..0.091 rows=178 loops=1)
         Index Cond: ((state)::text = 'California'::text)
         Buffers: shared read=2
 Planning:
   Buffers: shared hit=19 read=1
 Planning Time: 0.234 ms
 Execution Time: 0.139 ms
(11 rows)
```
Как видно - индекс хорошо ускоряет наш запрос. Сначала создается битовая карта, а потом с ее помощью производится выборка.
Самое главное что в данной выборке можно заметить - это сокращение времени выборки почти в 2.5 раза - с 0.345 ms
 до 0.139 ms
.

**Полнотекстовый поиск.**
7. Теперь предположим, что мы хотим найти все колледжи, у которых в названии есть слово 'University' - оценим результаты подобного поиска (через Like).
```
postgres=# EXPLAIN (ANALYZE, BUFFERS) select * from collages where name like '%University%';
                                                QUERY PLAN
----------------------------------------------------------------------------------------------------------
 Seq Scan on collages  (cost=0.00..92.68 rows=2135 width=57) (actual time=0.012..0.640 rows=2056 loops=1)
   Filter: ((name)::text ~~ '%University%'::text)
   Rows Removed by Filter: 1838
   Buffers: shared hit=44
 Planning Time: 0.081 ms
 Execution Time: 0.944 ms
(6 rows)
```
8. Создадим полнотекстовый индекс, используя функцию to_tsvector она преобразует значение в специальный набор хешей, удобный для быстрого поиска.
```
CREATE INDEX idx_collages_name ON collages USING GIN(to_tsvector('english', name));
```
9. Проверим работу нашего индекса.
```
postgres=# EXPLAIN (ANALYZE, BUFFERS) SELECT * FROM collages WHERE to_tsvector('english', name) @@ to_tsquery('english','University');
                                                          QUERY PLAN
-------------------------------------------------------------------------------------------------------------------------------
 Bitmap Heap Scan on collages  (cost=12.15..52.19 rows=19 width=57) (actual time=0.157..0.383 rows=2058 loops=1)
   Recheck Cond: (to_tsvector('english'::regconfig, (name)::text) @@ '''univers'''::tsquery)
   Heap Blocks: exact=44
   Buffers: shared hit=47
   ->  Bitmap Index Scan on idx_collages_name  (cost=0.00..12.15 rows=19 width=0) (actual time=0.145..0.145 rows=2058 loops=1)
         Index Cond: (to_tsvector('english'::regconfig, (name)::text) @@ '''univers'''::tsquery)
         Buffers: shared hit=3
 Planning:
   Buffers: shared hit=1
 Planning Time: 0.086 ms
 Execution Time: 0.474 ms
(11 rows)
```
Для работы с полнотекстовым индексом - требуется использовать функцию to_tsvector как на поле в бд, так и на искомую строку. Для поиска включений искомого слова (или его формы) - используется оператор `@@`. Как видим - поск так же заметно ускорился.
Данный индекс позволяет делать более сложные и точные выборки и может сильно повысить эффективность поиска в больших таблицах.

10. **Индекс на несколько полей.**
Для продолжения эксперемента - удалил все ранее созданные индексы.
```
drop index idx_collage_state;
drop index idx_collages_name;
```

11. Предположим, что мы выяснили, что часто требуется выборка колледжа сразу по трем полям - state, county и city - и мы хотим как-то ускорить этот поиск.
```
postgres=# EXPLAIN (ANALYZE, BUFFERS) select * from collages where state = 'California' and county = 'San Francisco' and city = 'San Francisco';
                                                                QUERY PLAN

------------------------------------------------------------------------------------------------------------------------
------------------
 Seq Scan on collages  (cost=0.00..112.15 rows=1 width=57) (actual time=0.020..0.491 rows=10 loops=1)
   Filter: (((state)::text = 'California'::text) AND ((county)::text = 'San Francisco'::text) AND ((city)::text = 'San F
rancisco'::text))
   Rows Removed by Filter: 3884
   Buffers: shared hit=44
 Planning Time: 0.539 ms
 Execution Time: 0.506 ms
```
12. Создаем индекс.
```
create index idx_collages_state_county_city ON collages (state, county, city);
```
13. Результат после создания индекса.
```
postgres=# EXPLAIN (ANALYZE, BUFFERS) select * from collages where state = 'California' and county = 'San Francisco' and city = 'San Francisco';
                                                                  QUERY PLAN
----------------------------------------------------------------------------------------------------------------------------------------------
 Index Scan using idx_collages_state_county_city on collages  (cost=0.28..8.30 rows=1 width=57) (actual time=0.020..0.026 rows=10 loops=1)
   Index Cond: (((state)::text = 'California'::text) AND ((county)::text = 'San Francisco'::text) AND ((city)::text = 'San Francisco'::text))
   Buffers: shared hit=6
 Planning Time: 0.078 ms
 Execution Time: 0.039 ms
(5 rows)
```
Будем честны - при наличии индекса чисто по State - выборка была бы немного медленней, но не критично. Составной индекс нужен, если выборка по данному набору полей будет происходить очень часто.

11. **Реализовать индекс на часть таблицы.**
Предположим, что индекс по наименованию колледжа нам нужен только в определенных случах. Например, если штат - 'California' - сделать это так же можно, создав индекс только на чать данных таблицы.
_В реальности - обычно такие индексы создаются не по штатам, или подобным данным - а по более техническим полям, по типу статуса, или типа._

```
CREATE INDEX idx_collages_name ON collages USING GIN(to_tsvector('english', name)) where state = 'California';
```

Оценим результаты поиска с без фильтра по штату.
```
postgres=# EXPLAIN (ANALYZE, BUFFERS) SELECT * FROM collages WHERE to_tsvector('english', name) @@ to_tsquery('english','University');
                                                QUERY PLAN
-----------------------------------------------------------------------------------------------------------
 Seq Scan on collages  (cost=0.00..1066.18 rows=19 width=57) (actual time=0.015..15.005 rows=2058 loops=1)
   Filter: (to_tsvector('english'::regconfig, (name)::text) @@ '''univers'''::tsquery)
   Rows Removed by Filter: 1836
   Buffers: shared hit=44
 Planning:
   Buffers: shared hit=17
 Planning Time: 0.137 ms
 Execution Time: 15.111 ms
(8 rows)

postgres=# EXPLAIN (ANALYZE, BUFFERS) SELECT * FROM collages WHERE to_tsvector('english', name) @@ to_tsquery('english','University');
```
И с фильтром по штату.
```
postgres=# EXPLAIN (ANALYZE, BUFFERS) SELECT * FROM collages WHERE state='California' and to_tsvector('english', name) @@ to_tsquery('english','University');
                                                               QUERY PLAN
----------------------------------------------------------------------------------------------------------------------------------------
 Bitmap Heap Scan on collages  (cost=8.00..12.27 rows=1 width=57) (actual time=0.024..0.039 rows=112 loops=1)
   Recheck Cond: ((to_tsvector('english'::regconfig, (name)::text) @@ '''univers'''::tsquery) AND ((state)::text = 'California'::text))
   Heap Blocks: exact=4
   Buffers: shared hit=6
   ->  Bitmap Index Scan on idx_collages_name  (cost=0.00..8.00 rows=1 width=0) (actual time=0.017..0.017 rows=112 loops=1)
         Index Cond: (to_tsvector('english'::regconfig, (name)::text) @@ '''univers'''::tsquery)
         Buffers: shared hit=2
 Planning:
   Buffers: shared hit=1
 Planning Time: 0.200 ms
 Execution Time: 0.067 ms
(11 rows)
```
Как видно - разница существенная.
Индексы на часть таблицы удобно использовать, если определенные выборки часто делаются только на определенном статусе, или на даные определенного типа - это позволит сократить объем хранимых данных, а так же время их сохранения, но при этом сохранить необходимую скорость в требуемых случаях.
