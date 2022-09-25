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
