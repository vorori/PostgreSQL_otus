# 1)
<pre>
создать новый проект в Google Cloud Platform, например postgres2023-, где yyyymm год, месяц вашего рождения (имя проекта должно быть уникально на уровне GCP), Яндекс облако или на любых ВМ, докере

--прошел регистрацию Yandex Compute Cloud 
</pre>

# 2)
<pre>
далее создать инстанс виртуальной машины с дефолтными параметрами
--создал centos 
</pre>

# 3)
<pre>
добавить свой ssh ключ в metadata ВМ
 
--добавил ключ работаю из cmd win 10
--ssh-keygen -t ed25519
</pre>

# 4)
<pre>
зайти удаленным ssh (первая сессия), не забывайте про ssh-add

--выполнил подключение к VM

ssh vorori@130.193.53.190
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
</pre>

# 5)
<pre>
поставить PostgreSQL 15

--пришел сюда
--https://www.postgresql.org/download/linux/redhat/

--установил ключи для репозитория
--Install the repository RPM:
--sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm

--установил 15 postgres 
--sudo yum install postgresql15-contrib



--инициализирую кластер

--sudo /usr/pgsql-15/bin/postgresql-15-setup initdb
Initializing database ... OK

--добавил сервер в автозагрузку
--sudo systemctl enable --now postgresql-15
--systemctl status postgresql-15
</pre>

# 6)
<pre>
зайти вторым ssh (вторая сессия)
--зашел 
--sudo su - postgres
</pre>

# 7)
<pre>
запустить везде psql из под пользователя postgres

--CMD
--ssh vorori@MY_IP_ADDR
--ssh vorori@130.193.53.190

--выполнено!
</pre>

# 8)
<pre>
выключить auto commit

--\set AUTOCOMMIT OFF
--postgres=# \echo :AUTOCOMMIT
--OFF

</pre>

# 9)
<pre>
сделать в первой сессии новую таблицу и наполнить ее данными 
create table persons(id serial, first_name text, second_name text);
insert into persons(first_name, second_name) values('ivan', 'ivanov'); 
insert into persons(first_name, second_name) values('petr', 'petrov'); 
commit;

--готово
</pre>

# 10)
<pre>
посмотреть текущий уровень изоляции: show transaction isolation level

--postgres=# show transaction isolation level;
 transaction_isolation
-----------------------
 read committed
(1 row)
</pre>

# 11)
<pre>
начать новую транзакцию в обоих сессиях с дефолтным (не меняя) уровнем изоляции

--BEGIN;
--BEGIN;
</pre>

# 12)
<pre>
в первой сессии добавить новую запись 
insert into persons(first_name, second_name) values('sergey', 'sergeev');

--INSERT 0 1
</pre>

# 13)
<pre>
сделать select * from persons во второй сессии


--postgres=*# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
(2 rows)
</pre>

# 14)
<pre>
видите ли вы новую запись и если да то почему?

--Новую запись не видно так как auto commit отключен
--транзакция не зафиксированна
</pre>

# 15)
<pre>
завершить первую транзакцию - commit;

--postgres=*# commit;
COMMIT
</pre>

# 16)
<pre>
сделать select * from persons во второй сессии

--postgres=*# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 rows)
</pre>

# 17)
<pre>
видите ли вы новую запись и если да то почему?

 --Новая запись появилась т.к --выполнилась транзакция в --первой сессии мы --зафиксировали изменение.
</pre>


# 18)
<pre>
завершите транзакцию во второй сессии

--postgres=*# commit;
--COMMIT
</pre>

# 19)
<pre>
начать новые но уже repeatable read транзации
set transaction isolation level repeatable read;

--BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
--postgres=*# show transaction isolation level;
 transaction_isolation
-----------------------
 repeatable read
(1 row)
</pre>

# 20)
<pre>
в первой сессии добавить новую запись 
insert into persons(first_name, second_name) values('sveta', 'svetova');


ERROR:  column "sveta" does not exist
--исправил запрос insert 
insert into persons(first_name, second_name) values('sveta', 'svetova');
INSERT 0 1
</pre>

# 21)
<pre>
сделать select * from persons во второй сессии

--postgres=# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 rows)
</pre>

# 22)
<pre>
видите ли вы новую запись и если да то почему?

Новую запись не видно, фиксация транзакции первой сессии не произведена.
</pre>

# 23)
<pre>
завершить первую транзакцию - commit;

--postgres=*# commit;
COMMIT
</pre>

# 24)
<pre>
сделать select * from persons во второй сессии

--postgres=# select * from --persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
</pre>


# 25)
<pre>
видите ли вы новую запись и если да то почему?

--Новую запись не видно, фантомное чтение при repeatable --read не допускается.
</pre>

# 26)
<pre>
завершить вторую транзакцию

--postgres=*# commit;
COMMIT
</pre>

# 27)
<pre>
сделать select * from persons во второй сессии

--postgres=*# select * from --persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
  6 | sveta      | svetova
(4 rows)
</pre>

# 28)
<pre>
видите ли вы новую запись и если да то почему? 

--новая строка отобразилась, поскольку при выполнении commit --мы зафиксировали нашу транзакцию и произошла смена с LEVEL --REPEATABLE READ на LEVEL READ COMMITTED.
</pre>