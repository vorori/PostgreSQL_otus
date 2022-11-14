# 1    
создайте новый кластер PostgresSQL 14 
создал

systemctl status postgresql-14
[root@postgre vorori]# systemctl status postgresql-14
● postgresql-14.service - PostgreSQL 14 database server
   Loaded: loaded (/usr/lib/systemd/system/postgresql-14.service; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2022-11-14 06:24:10 UTC; 2min 58s ago
     Docs: https://www.postgresql.org/docs/14/static/
  Process: 860 ExecStartPre=/usr/pgsql-14/bin/postgresql-14-check-db-dir ${PGDATA} (code=exited, status=0/SUCCESS)
 Main PID: 874 (postmaster)
   CGroup: /system.slice/postgresql-14.service
           ├─ 874 /usr/pgsql-14/bin/postmaster -D /var/lib/pgsql/14/data/
           ├─1006 postgres: logger
           ├─1008 postgres: checkpointer
           ├─1009 postgres: background writer
           ├─1010 postgres: walwriter
           ├─1011 postgres: autovacuum launcher
           ├─1012 postgres: stats collector
           └─1013 postgres: logical replication launcher

# 2 
зайдите в созданный кластер под пользователем postgres

[root@postgre vorori]# su - postgres
-bash-4.2$ psql
psql (14.6)
Type "help" for help.
postgres=#

# 3 
создайте новую базу данных testdb
create database testdb;

# 4 
зайдите в созданную базу данных под пользователем postgres

\c testdb

# 5 
создайте новую схему testnm

testdb=# CREATE SCHEMA testnm;
CREATE SCHEMA

# 6 
создайте новую таблицу t1 с одной колонкой c1 типа integer


CREATE TABLE t1(c1 integer);
CREATE TABLE


# 7 
вставьте строку со значением c1=1

INSERT INTO t1 VALUES (1);

# 8 
создайте новую роль readonly

CREATE ROLE readonly;

# 9 
дайте новой роли право на подключение к базе данных testdb

GRANT CONNECT ON DATABASE testdb to readonly;

# 10 
дайте новой роли право на использование схемы testnm

GRANT USAGE ON SCHEMA testnm to readonly;

# 11 
дайте новой роли право на select для всех таблиц схемы testnm

testdb=# GRANT SELECT ON ALL TABLES IN SCHEMA testnm to readonly;
GRANT

# 12 
создайте пользователя testread с паролем test123

testdb=# CREATE USER testread WITH PASSWORD 'test123';
CREATE ROLE

# 13 
дайте роль readonly пользователю testread

GRANT readonly TO testread;

# 14 
зайдите под пользователем testread в базу данных testdb

добавил в pg_hba.conf
local   all             all                                     scram-sha-256
select pg_reload_conf();

psql -U testread -d testdb -h localhost

или

\c testdb testread
Password for user testread:
You are now connected to database "testdb" as user "testread".
testdb=>


# 15 
сделайте select * from t1;

testdb=> select * from t1;
ERROR:  permission denied for table t1

# 16 
получилось? (могло если вы делали сами не по шпаргалке и не упустили один существенный момент про который позже)

нету прав на таблицу потому что она находится в схеме GRANT SELECT владелец этой таблицы postgres . 
мы не предоставляли GRANT SELECT на все таблицы в схеме public для роли readonly

# 17 
напишите что именно произошло в тексте домашнего задания
при коннекте к базе мы работаем в схеме public мы не давали роли readonly доступ на t1 и соответственно пользователь testread
тоже не имеет доступа к этой таблице

# 18 
у вас есть идеи почему? ведь права то дали?

права давали на другую схему где нет этой таблицы 

# 19 
посмотрите на список таблиц

testdb=> \dt
        List of relations
 Schema | Name | Type  |  Owner
--------+------+-------+----------
 public | t1   | table | postgres
(1 row)

# 20 
подсказка в шпаргалке под пунктом 20


# 21 
а почему так получилось с таблицей (если делали сами и без шпаргалки то может у вас все нормально)
права давали на другую схему где нет этой таблицы 

testdb=> select * from t1;
ERROR:  permission denied for table t1

# 22 
вернитесь в базу данных testdb под пользователем postgres

вернулся

# 23 
удалите таблицу t1

testdb= drop table t1;
DROP TABLE

# 24 
создайте ее заново но уже с явным указанием имени схемы testnm


CREATE TABLE testnm.t1(c1 integer);
CREATE TABLE

# 25 
вставьте строку со значением c1=1

testdb=# INSERT INTO testnm.t1 VALUES (1);
INSERT 0 1

# 26 
зайдите под пользователем testread в базу данных testdb

вернулся

# 27 
сделайте select * from testnm.t1;

testdb=> select * from t1;
ERROR:  permission denied for table t1

# 28 
получилось?

доступа нет

# 29 
есть идеи почему? если нет - смотрите шпаргалку

доступа нет потому что таблицу мы создали после выдачи прав на схему
мы давали права на схему когда небыло этой таблицы 
чтобы таблица была доступна надо выдавать права на select all table после того как таблица создана


# 30 
как сделать так чтобы такое больше не повторялось? если нет идей - смотрите шпаргалку

можно использовать привелегии по умолчанию чтобы гранты применялись автоматически при 
создании нового обекта в нашем случае таблицы

ALTER DEFAULT privileges in SCHEMA testnm grant SELECT on TABLEs to readonly;
ALTER DEFAULT PRIVILEGES

testdb=# \ddp
            Default access privileges
  Owner   | Schema | Type  |  Access privileges
----------+--------+-------+---------------------
 postgres | testnm | table | readonly=r/postgres
(1 row)


# 31 
testdb=> select * from t1;
ERROR:  permission denied for table t1

# 32 
получилось?

снова нет прав на просмотр потому что привелегии по умолчанию будут работать токо после создания нового обекта
а эта таблица у нас уже создана до применения привелегий по умолчанию

снова даю явную привелгию
GRANT SELECT ON ALL TABLES IN SCHEMA testnm to readonly;


# 33 
есть идеи почему? если нет - смотрите шпаргалку


# 31 
сделайте select * from testnm.t1;

testdb=> select * from testnm.t1;
 c1
----
  1
(1 row)

# 32 
получилось?

да

# 33 
ура!

# 34 
теперь попробуйте выполнить команду create table t2(c1 integer); insert into t2 values (2);

testdb=> SET search_path to public;
SET
testdb=> create table t2(c1 integer);
CREATE TABLE
testdb=> insert into t2 values (2);
INSERT 0 1

# 35 
а как так? нам же никто прав на создание таблиц и insert в них под ролью readonly?

надо явно запретить 


# 36 
есть идеи как убрать эти права? если нет - смотрите шпаргалку

надо явно запретить для роли public создание обектов и забрать все привелегии на базу данных у роли public
чтобы вновь созданные пользователи не могли подключаться к базе testdb и соответственно видить какие обекты там есть

REVOKE CREATE on SCHEMA public FROM public; 
REVOKE all on DATABASE testdb FROM public; 


# 37 
если вы справились сами то расскажите что сделали и почему, если смотрели шпаргалку - объясните что сделали и почему выполнив указанные в ней команды

запретил для роли public создание обектов и забрал все привелегии на базы данных у роли public.(права роли public явно включены во все роли)
чтобы вновь созданные пользователи не могли подключаться к базе testdb

# 38 
теперь попробуйте выполнить команду create table t3(c1 integer); insert into t2 values (2);

testdb=>create table t3(c1 integer);
ERROR:  permission denied for schema public

testdb=> insert into t2 values (2);
INSERT 0 1

# 39 
расскажите что получилось и почему

создать новую таблицу под ролью testread теперь не получается потому что забрали явные права у всеобшей роли public
а вставка данных проходит потому что таблица t2 была создана до отзыва привелегий у роли public и к томуже владелец 
этой таблицы роль testread и соответственно вставка данных будет работать 


testdb=> \dt+
                                      List of relations
 Schema |  Name  | Type  |  Owner   | Persistence | Access method |    Size    | Description
--------+--------+-------+----------+-------------+---------------+------------+-------------
 public | t2     | table | testread | permanent   | heap          | 8192 bytes |