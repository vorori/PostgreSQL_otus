1)ю
--------------------------------------------------------------------------------------------------------------
создать новый проект в Google Cloud Platform, например postgres2022-, где yyyymm год, месяц вашего рождения (имя проекта должно быть уникально на уровне GCP), Яндекс облако или на любых ВМ, докере

--прошел регистрацию Yandex Compute Cloud 
--------------------------------------------------------------------------------------------------------------

2)
--------------------------------------------------------------------------------------------------------------
далее создать инстанс виртуальной машины с дефолтными параметрами

--создал centos 7 name postgres1989-03
--------------------------------------------------------------------------------------------------------------

3)
--------------------------------------------------------------------------------------------------------------
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

--------------------------------------------------------------------------------------------------------------

4)
--------------------------------------------------------------------------------------------------------------
зайти удаленным ssh (первая сессия), не забывайте про ssh-add

--выполнил подключение к VM

C:\Users\vorori>ssh vorori@51.250.100.46
The authenticity of host '51.250.100.46 (51.250.100.46)' can't be established.
ECDSA key fingerprint is SHA256:QSN1OxfkkfwR7knm062VLu+ylJFRrLr*********************.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes

5)
--------------------------------------------------------------------------------------------------------------
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
--------------------------------------------------------------------------------------------------------------


6)
--------------------------------------------------------------------------------------------------------------
зайти вторым ssh (вторая сессия)

--зашел добавил сервер в автозагрузку

--sudo systemctl enable --now postgresql-14
--systemctl status postgresql-14
--sudo su - postgres
--------------------------------------------------------------------------------------------------------------

7)
--------------------------------------------------------------------------------------------------------------
запустить везде psql из под пользователя postgres

--CMD
--ssh vorori@MY_IP_ADDR

--выполнено!
--------------------------------------------------------------------------------------------------------------


выключить auto commit

сделать в первой сессии новую таблицу и наполнить ее данными create table persons(id serial, first_name text, second_name text); insert into persons(first_name, second_name) values('ivan', 'ivanov'); insert into persons(first_name, second_name) values('petr', 'petrov'); commit;

посмотреть текущий уровень изоляции: show transaction isolation level

начать новую транзакцию в обоих сессиях с дефолтным (не меняя) уровнем изоляции

в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sergey', 'sergeev');

сделать select * from persons во второй сессии

видите ли вы новую запись и если да то почему?

завершить первую транзакцию - commit;

сделать select * from persons во второй сессии

видите ли вы новую запись и если да то почему?

завершите транзакцию во второй сессии

начать новые но уже repeatable read транзации - set transaction isolation level repeatable read;

в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sveta', 'svetova');

сделать select * from persons во второй сессии

видите ли вы новую запись и если да то почему?

завершить первую транзакцию - commit;

сделать select * from persons во второй сессии

видите ли вы новую запись и если да то почему?

завершить вторую транзакцию

сделать select * from persons во второй сессии

видите ли вы новую запись и если да то почему? ДЗ сдаем в виде миниотчета в markdown в гите