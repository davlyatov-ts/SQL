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









