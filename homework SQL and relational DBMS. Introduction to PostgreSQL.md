# 1)
создать новый проект в Google Cloud Platform, например postgres2022-, где yyyymm год, месяц вашего рождения (имя проекта должно быть уникально на уровне GCP), Яндекс облако или на любых ВМ, докере

--прошел регистрацию Yandex Compute Cloud 


# 2)
далее создать инстанс виртуальной машины с дефолтными параметрами

--создал centos 7 name postgres1989-03


# 3)
добавить свой ssh ключ в metadata ВМ
 
--добавил ключ работаю из cmd win 10
--ssh-keygen -t ed25519

Generating public/private ed25519 key pair.
Enter file in which to save the key (C:\Users\vorori/.ssh/id_*********):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in C:\Users\vorori/.ssh/id_**********.
Your public key has been saved in C:\Users\vorori/.ssh/id_*******.pub.
The key fingerprint is:
SHA256:n8ryIT32oLtc6k/ukIeZaOjckd/52****************** vorori@adm
The key's randomart image is:
+--[ED25519 256]--+
|. . .            |
|.o o             |
|o.E              |
|..               |
|  .     S        |
|   o o * + .     |
|  . * B X O      |
| o o *.& ^       |
| ***********     |
+----[SHA256]-----+


# 4)
зайти удаленным ssh (первая сессия), не забывайте про ssh-add

--выполнил подключение к VM

C:\Users\vorori>ssh vorori@51.250.100.46
The authenticity of host '51.250.100.46 (51.250.100.46)' can't be established.
ECDSA key fingerprint is SHA256:QSN1OxfkkfwR7knm062VLu+ylJFRrLr*********************.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes

# 5)
поставить PostgreSQL

--пришел сюда
--https://www.postgresql.org/download/linux/redhat/

--установил ключи для репозитория
--Install the repository RPM:
--sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm

--установил 14 postgres (вот такой оригинальной командой :) )
--sudo yum install postgresql14-contrib


Install  1 Package (+36 Dependent packages)
Total download size: 36 M
Installed size: 142 M
Is this ok [y/d/N]: y



--инициализирую кластер

--sudo /usr/pgsql-14/bin/postgresql-14-setup initdb
Initializing database ... OK



# 6)
зайти вторым ssh (вторая сессия)

--зашел добавил сервер в автозагрузку

--sudo systemctl enable --now postgresql-14
--systemctl status postgresql-14
--sudo su - postgres


# 7)
запустить везде psql из под пользователя postgres

--CMD
--ssh vorori@MY_IP_ADDR

--выполнено!



# 8)
выключить auto commit

--\set AUTOCOMMIT OFF
--postgres=# \echo :AUTOCOMMIT
--OFF

# 9)
сделать в первой сессии новую таблицу и наполнить ее данными create table persons(id serial, first_name text, second_name text); insert into persons(first_name, second_name) values('ivan', 'ivanov'); insert into persons(first_name, second_name) values('petr', 'petrov'); commit;

--готово

# 10)
посмотреть текущий уровень изоляции: show transaction isolation level

--postgres=# show transaction --isolation level;
 transaction_isolation
-----------------------
 read committed
(1 row)

# 11)

начать новую транзакцию в обоих сессиях с дефолтным (не меняя) уровнем изоляции

--BEGIN;
--BEGIN;

# 12)

в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sergey', 'sergeev');

--INSERT 0 1


# 13)
сделать select * from persons во второй сессии


--postgres=*# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
(2 rows)



# 14)
видите ли вы новую запись и если да то почему?

--Новую запись не видно так как auto commit отключен


# 15)
завершить первую транзакцию - commit;

--postgres=*# commit;
COMMIT

# 16)
сделать select * from persons во второй сессии

--postgres=*# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 rows)

# 17)
видите ли вы новую запись и если да то почему?

 --Новая запись появилась т.к --выполнилась транзакция в --первой сессии мы --зафиксировали изменение.

# 18)
завершите транзакцию во второй сессии

--postgres=*# commit;
--COMMIT

# 19)
начать новые но уже repeatable read транзации - set transaction isolation level repeatable read;

--BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
--postgres=*# show transaction --isolation level;
 transaction_isolation
-----------------------
 repeatable read
(1 row)


# 20)
в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sveta', 'svetova');


ERROR:  column "sveta" does not exist
--исправил запрос insert 
insert into persons(first_name, second_name) values('sveta', 'svetova');
INSERT 0 1

# 21)
сделать select * from persons во второй сессии

--postgres=# select * from --persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 rows)


# 22)
видите ли вы новую запись и если да то почему?

Новую запись не видно, фиксация транзакции первой сессии не произведена.

# 23)
завершить первую транзакцию - commit;

--postgres=*# commit;
COMMIT


# 24)
сделать select * from persons во второй сессии

--postgres=# select * from --persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev

# 25)
видите ли вы новую запись и если да то почему?

--Новую запись не видно, фантомное чтение при repeatable --read не допускается.

26)
завершить вторую транзакцию

--postgres=*# commit;
COMMIT

# 27)
сделать select * from persons во второй сессии

--postgres=*# select * from --persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
  6 | sveta      | svetova
(4 rows)


# 28)
видите ли вы новую запись и если да то почему? ДЗ сдаем в виде миниотчета в markdown в гите

--новая строка отобразилась, поскольку при выполнении commit --мы зафиксировали нашу транзакцию и произошла смена с LEVEL --REPEATABLE READ на LEVEL READ COMMITTED.