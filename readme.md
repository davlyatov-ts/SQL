Используя docker поднимите инстанс PostgreSQL (версию 12) c 2 volume, в который будут складываться данные БД и бэкапы.

Приведите получившуюся команду или docker-compose манифест.

```
alex@AlexPC:~/Docker$ docker volume ls
DRIVER    VOLUME NAME
local     4e89e95e54f6ff7466c7ae7f4c98bdb8d1c4b52f999d90a94be26bba933ea72a
local     8a65abb70e0730628059bf785af53c699ab909a09991a36af6d76327e7c454d6
local     volume_1
local     volume_2
alex@AlexPC:~/Docker$ docker run \
--name postgres \
-p 5432:5432 \
-v volume_1:/data/volume1 -v volume_2:/data/volume2 \
-e POSTGRES_USER=test-admin-user \
-e POSTGRES_PASSWORD=security \
-e POSTGRES_DB=test_db -d postgres:12
alex@AlexPC:~/Docker$ docker exec -it postgres /bin/bash
root@d21a2380e245:/# ls -l /data
total 8
drwxr-xr-x 2 root root 4096 Jun  6 18:38 volume1
drwxr-xr-x 2 root root 4096 Jun  6 18:38 volume2
root@d21a2380e245:/# 
```
_______________________________________

#Задача 2

В БД из задачи 1:

- создайте пользователя test-admin-user и БД test_db
- в БД test_db создайте таблицу orders и clients (спeцификация таблиц ниже)
- предоставьте привилегии на все операции пользователю test-adminuser на таблицы БД test_db
- создайте пользователя test-simple-user
- предоставьте пользователю test-simple-user права на SELECT/INSERT/UPDATE/DELETE данных таблиц БД test_db<br>

Таблица orders:<br>

- id (serial primary key)
- наименование (string)
- цена (integer)

Таблица clients:

- id (serial primary key)
- фамилия (string)
- страна проживания (string, index)
- заказ (foreign key orders)

Приведите:

- итоговый список БД после выполнения пунктов выше,

```
test_db-# \d
                  List of relations
 Schema |      Name      |   Type   |      Owner      
--------+----------------+----------+-----------------
 public | clients        | table    | test-admin-user
 public | clients_id_seq | sequence | test-admin-user
 public | orders         | table    | test-admin-user
 public | orders_id_seq  | sequence | test-admin-user
(4 rows)
```
- описание таблиц (describe)

```
select table_name, column_name, data_type from information_schema.columns where table_name = 'orders'
Name       |Value  |
-----------+-------+
table_name |orders |
column_name|id     |
data_type  |integer|

select table_name, column_name, data_type from information_schema.columns where table_name = 'clients'
Name       |Value  |
-----------+-------+
table_name |clients|
column_name|id     |
data_type  |integer|
=====================================================================================================================================================
test_db=# \d+ orders
                                                     Table "public.orders"
   Column   |     Type      | Collation | Nullable |              Default               | Storage  | Stats target | Description 
------------+---------------+-----------+----------+------------------------------------+----------+--------------+-------------
 id         | integer       |           | not null | nextval('orders_id_seq'::regclass) | plain    |              | 
 order_name | character(20) |           |          |                                    | extended |              | 
 order_cost | integer       |           |          |                                    | plain    |              | 
Indexes:
    "orders_pkey" PRIMARY KEY, btree (id)
Referenced by:
    TABLE "clients" CONSTRAINT "clients_client_order_fkey" FOREIGN KEY (client_order) REFERENCES orders(id)
Access method: heap
=====================================================================================================================================================
test_db=# \d+ clients
                                                        Table "public.clients"
     Column      |     Type      | Collation | Nullable |               Default               | Storage  | Stats target | Description 
-----------------+---------------+-----------+----------+-------------------------------------+----------+--------------+-------------
 id              | integer       |           | not null | nextval('clients_id_seq'::regclass) | plain    |              | 
 client_lastname | character(20) |           |          |                                     | extended |              | 
 client_country  | character(20) |           |          |                                     | extended |              | 
 client_order    | integer       |           |          |                                     | plain    |              | 
Indexes:
    "clients_pkey" PRIMARY KEY, btree (id)
    "country" UNIQUE, btree (client_country)
Foreign-key constraints:
    "clients_client_order_fkey" FOREIGN KEY (client_order) REFERENCES orders(id)
Access method: heap

test_db=# 
=====================================================================================================================================================
```

- SQL-запрос для выдачи списка пользователей с правами над таблицами test_db<br>

```
select * from pg_catalog.pg_user
```
- список пользователей с правами над таблицами test_db<br>

```
test_db-# \dp
                                                Access privileges
 Schema |      Name      |   Type   |              Access privileges              | Column privileges | Policies 
--------+----------------+----------+---------------------------------------------+-------------------+----------
 public | clients        | table    | "test-admin-user"=arwdDxt/"test-admin-user"+|                   | 
        |                |          | "test-simple-user"=arwd/"test-admin-user"   |                   | 
 public | clients_id_seq | sequence |                                             |                   | 
 public | orders         | table    | "test-admin-user"=arwdDxt/"test-admin-user"+|                   | 
        |                |          | "test-simple-user"=arwd/"test-admin-user"   |                   | 
 public | orders_id_seq  | sequence |                                             |                   | 
(4 rows)
```

#Задача 3

Используя SQL синтаксис - наполните таблицы следующими тестовыми данными:<br>

Таблица orders<br>

|Наименование|цена|
|------------|----|
|Шоколад     |10  |
|Принтер     |3000|
|Книга       |500 |
|Монитор     |7000|
|Гитара      |4000|

Таблица clients<br>


|    ФИО    |Страна проживания|
|-----------|-----------------|
|Иванов Иван Иванович|USA     |
|Петров Петр Петрович|Canada  |
|Иоганн Себастьян Бах|Japan   |
|Ронни Джеймс Дио    |Russia  |
|Ritchie Blackmore   |Russia  |

Используя SQL синтаксис:<br>

- вычислите количество записей для каждой таблицы<br>

```
select count (id) from clients
Name |Value|
-----+-----+
count|5    |

select count (id) from orders
Name |Value|
-----+-----+
count|5    |
```
- приведите в ответе:
	- запросы
	- результаты их выполнения.


#Задача 4

Часть пользователей из таблицы clients решили оформить заказы из таблицы orders.<br>

Используя foreign keys свяжите записи из таблиц, согласно таблице:<br>

|    ФИО    |Заказ|
|-----------|-----|
|Иванов Иван Иванович|Книга|
|Петров Петр Петрович|Монитор|
|Иоганн Себастьян Бах|Гитара|

Приведите SQL-запросы для выполнения данных операций.<br>
```
update clients set client_order = 3 where client_lastname = 'Иванов Иван Иванович'
update clients set client_order = 4 where client_lastname = 'Петров Петр Петрович'
update clients set client_order = 5 where client_lastname = 'Иоганн Себастьян Бах'
```

Приведите SQL-запрос для выдачи всех пользователей, которые совершили заказ, а также вывод данного запроса.<br>

```
select * from clients where client_order is not null

id|client_lastname     |client_country      |client_order
--+--------------------+--------------------+------------
 1|Иванов Иван Иванович|USA                 |           3
 2|Петров Петр Петрович|Canada              |           4
 3|Иоганн Себастьян Бах|Japan               |           5
```

Подсказк - используйте директиву ``UPDATE``.
_____________________________

#Задача 5

Получите полную информацию по выполнению запроса выдачи всех пользователей из задачи 4 (используя директиву EXPLAIN).<br>

```
explain select * from clients where client_order is not null


QUERY PLAN                                             |
-------------------------------------------------------+
Seq Scan on clients  (cost=0.00..1.05 rows=5 width=176)|
  Filter: (client_order IS NOT NULL)                   |
```
`cost` - Приблизительная стоимость запуска. Это время, которое проходит, прежде чем начнётся этап вывода данных, например для сортирующего узла это время сортировки.<br>

rows - Ожидаемое число строк, которое должен вывести этот узел плана. При этом так же предполагается, что узел выполняется до конца.<br>

width - Ожидаемый средний размер строк, выводимых этим узлом плана (в байтах).<br>

Общая оценка "стоимости" запроса по времени и ресурсам. Позволяет оценить насколько корректно и точно создан запрос.<br>

Приведите получившийся результат и объясните что значат полученные значения.<br>

#Задача 6

Создайте бэкап БД test_db и поместите его в volume, предназначенный для бэкапов (см. Задачу 1).<br>

Остановите контейнер с PostgreSQL (но не удаляйте volumes).<br>

Поднимите новый пустой контейнер с PostgreSQL.<br>

Восстановите БД test_db в новом контейнере.<br>

Приведите список операций, который вы применяли для бэкапа данных и восстановления.<br>
```
root@32d7de5c539f:/# pg_dump test_db -U test-admin-user > /data/volume1/test_db.sql
root@32d7de5c539f:/# exit
exit
lex@userver:~/Docker$ docker volume list
DRIVER    VOLUME NAME
local     37685cbaa0614c981ab4a0795c184a191ac2df20c63c3ce7634380feb48ec3f4
local     volume1
local     volume2
lex@userver:~/Docker$

lex@userver:~/Docker$ docker stop postgres
postgres

lex@userver:~/Docker$ docker ps -a
CONTAINER ID   IMAGE               COMMAND                  CREATED         STATUS                     PORTS                                       NAMES
32d7de5c539f   postgres:12         "docker-entrypoint.s…"   2 hours ago     Exited (0) 1 minutes ago                                               postgres
0a81efbe8eda   pukoff/ansible:v1   "ansible-playbook --…"   2 weeks ago     Exited (0) 2 weeks ago                                                 serene_herschel
a800b719c367   debian:latest       "bash"                   2 weeks ago     Exited (137) 2 hours ago                                               debian
e44575b56f92   centos:latest       "/bin/bash"              2 weeks ago     Exited (0) 2 hours ago                                                 centos
lex@userver:~/Docker$ docker ps -a
CONTAINER ID   IMAGE               COMMAND                  CREATED       STATUS                     PORTS     NAMES
0a81efbe8eda   pukoff/ansible:v1   "ansible-playbook --…"   2 weeks ago   Exited (0) 2 weeks ago               serene_herschel
a800b719c367   debian:latest       "bash"                   2 weeks ago   Exited (137) 2 hours ago             debian
e44575b56f92   centos:latest       "/bin/bash"              2 weeks ago   Exited (0) 2 hours ago               centos

lex@userver:~/Docker$ docker run --name postgres -p 5432:5432 -v volume1:/data/volume1 -v volume2:/data/volume2 -e POSTGRES_USER=test-admin-user -e POSTGRES_PASSWORD=security -e POSTGRES_DB=test_db -d postgres:12
2e5c23f1cdfbbbf4e9411bf51a255adcbe7b07b8ceacbe3bf8c1eeccaee18439
lex@userver:~/Docker$ docker exec -it postgres /bin/bash
root@2e5c23f1cdfb:/# psql test_db -U test-admin-user < /data/volume1/test_db.sql 
SET
SET
SET
SET
SET
 set_config 
------------
 
(1 row)

SET
SET
SET
SET
SET
SET
CREATE TABLE
ALTER TABLE
CREATE SEQUENCE
ALTER TABLE
ALTER SEQUENCE
CREATE TABLE
ALTER TABLE
CREATE SEQUENCE
ALTER TABLE
ALTER SEQUENCE
ALTER TABLE
ALTER TABLE
COPY 5
COPY 5
 setval 
--------
      1
(1 row)

 setval 
--------
      1
(1 row)

ALTER TABLE
ALTER TABLE
CREATE INDEX
ALTER TABLE
root@2e5c23f1cdfb:/# psql test_db -U test-admin-user
psql (12.11 (Debian 12.11-1.pgdg110+1))
Type "help" for help.

test_db=# \dp *
                                                Access privileges
 Schema |      Name      |   Type   |              Access privileges              | Column privileges | Policies 
--------+----------------+----------+---------------------------------------------+-------------------+----------
 public | clients        | table    | "test-admin-user"=arwdDxt/"test-admin-user"+|                   | 
        |                |          | "test-simple-user"=arwd/"test-admin-user"   |                   | 
 public | clients_id_seq | sequence |                                             |                   | 
 public | orders         | table    | "test-admin-user"=arwdDxt/"test-admin-user"+|                   | 
        |                |          | "test-simple-user"=arwd/"test-admin-user"   |                   | 
 public | orders_id_seq  | sequence |                                             |                   | 
(4 rows)
```

Листинг SQL запросов<br>
```
create table orders (id SERIAL primary key, order_name CHAR(20), order_cost INTEGER)

create table clients (id SERIAL primary key, client_lastname CHAR(20), client_country CHAR(20), client_order INTEGER references orders(id))

create index country on clients (client_country)

create user "test-simple-user"

grant select,insert,update,delete on orders, clients to "test-simple-user"

select table_name, column_name, data_type from information_schema.columns where table_name = 'orders'

select table_name, column_name, data_type from information_schema.columns where table_name = 'clients'

insert into orders values (1,'Шоколад',10)
insert into orders values (2,'Принтер',3000)
insert into orders values (3,'Книга',500)
insert into orders values (4,'Монитор',7000)
insert into orders values (5,'Гитара',4000)


insert into clients values (2,'Петров Петр Петрович','Canada')
insert into clients values (3,'Иоганн Себастьян Бах','Japan')
insert into clients values (4,'Ронни Джеймс Дио','Russia')
insert into clients values (5,'Ritchie Blackmore','Russia')

select count (*) from orders
select count (*) from clients

update clients set client_order = 3 where client_lastname = 'Иванов Иван Иванович'
update clients set client_order = 4 where client_lastname = 'Петров Петр Петрович'
update clients set client_order = 5 where client_lastname = 'Иоганн Себастьян Бах'

explain select * from clients where client_order is not null
```










