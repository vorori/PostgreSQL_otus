1)
--------------------------------------------------------------------------------------------------------------
создать новый проект в Google Cloud Platform, например postgres2022-, где yyyymm год, месяц вашего рождения (имя проекта должно быть уникально на уровне GCP), Яндекс облако или на любых ВМ, докере

прошел регистрацию Yandex Compute Cloud 
--------------------------------------------------------------------------------------------------------------

2)
--------------------------------------------------------------------------------------------------------------
далее создать инстанс виртуальной машины с дефолтными параметрами

создал centos 7 name postgres1989-03
--------------------------------------------------------------------------------------------------------------

3)
добавить свой ssh ключ в metadata ВМ

добавил ключ работаю из cmd win 10
ssh-keygen -t ed25519

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


--------------------------------------------------------------------------------------------------------------
зайти удаленным ssh (первая сессия), не забывайте про ssh-add
выполнил подключение к VM

C:\Users\vorori>ssh vorori@51.250.100.46
The authenticity of host '51.250.100.46 (51.250.100.46)' can't be established.
ECDSA key fingerprint is SHA256:QSN1OxfkkfwR7knm062VLu+ylJFRrLr*********************.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes


--------------------------------------------------------------------------------------------------------------
поставить PostgreSQL

пришел сюда
https://www.postgresql.org/download/linux/redhat/

установил ключи для репозитория
# Install the repository RPM:
sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm

установил 14 postgres (вот такой оригинальной командой :) )
sudo yum install postgresql14-contrib


=======================================================================================================================================================================================
 Package                                                           Arch                                              Version                                                        Repository                                          Size
=======================================================================================================================================================================================
Installing:
 postgresql14-contrib                                              x86_64                                            14.5-1PGDG.rhel7                                               pgdg14                                             686 k
Installing for dependencies:
 libicu                                                            x86_64                                            50.2-4.el7_7                                                   base                                               6.9 M
 libxslt                                                           x86_64                                            1.1.28-6.el7                                                   base                                               242 k
 perl                                                              x86_64                                            4:5.16.3-299.el7_9                                             updates                                            8.0 M
 perl-Carp                                                         noarch                                            1.26-244.el7                                                   base                                                19 k
 perl-Encode                                                       x86_64                                            2.51-7.el7                                                     base                                               1.5 M
 perl-Exporter                                                     noarch                                            5.68-3.el7                                                     base                                                28 k
 perl-File-Path                                                    noarch                                            2.09-2.el7                                                     base                                                26 k
 perl-File-Temp                                                    noarch                                            0.23.01-3.el7                                                  base                                                56 k
 perl-Filter                                                       x86_64                                            1.49-3.el7                                                     base                                                76 k
 perl-Getopt-Long                                                  noarch                                            2.40-3.el7                                                     base                                                56 k
 perl-HTTP-Tiny                                                    noarch                                            0.033-3.el7                                                    base                                                38 k
 perl-PathTools                                                    x86_64                                            3.40-5.el7                                                     base                                                82 k
 perl-Pod-Escapes                                                  noarch                                            1:1.04-299.el7_9                                               updates                                             52 k
 perl-Pod-Perldoc                                                  noarch                                            3.20-4.el7                                                     base                                                87 k
 perl-Pod-Simple                                                   noarch                                            1:3.28-4.el7                                                   base                                               216 k
 perl-Pod-Usage                                                    noarch                                            1.63-3.el7                                                     base                                                27 k
 perl-Scalar-List-Utils                                            x86_64                                            1.27-248.el7                                                   base                                                36 k
 perl-Socket                                                       x86_64                                            2.010-5.el7                                                    base                                                49 k
 perl-Storable                                                     x86_64                                            2.45-3.el7                                                     base                                                77 k
 perl-Text-ParseWords                                              noarch                                            3.29-4.el7                                                     base                                                14 k
 perl-Time-HiRes                                                   x86_64                                            4:1.9725-3.el7                                                 base                                                45 k
 perl-Time-Local                                                   noarch                                            1.2300-2.el7                                                   base                                                24 k
 perl-constant                                                     noarch                                            1.27-2.el7                                                     base                                                19 k
 perl-libs                                                         x86_64                                            4:5.16.3-299.el7_9                                             updates                                            690 k
 perl-macros                                                       x86_64                                            4:5.16.3-299.el7_9                                             updates                                             44 k
 perl-parent                                                       noarch                                            1:0.225-244.el7                                                base                                                12 k
 perl-podlators                                                    noarch                                            2.5.1-3.el7                                                    base                                               112 k
 perl-threads                                                      x86_64                                            1.87-4.el7                                                     base                                                49 k
 perl-threads-shared                                               x86_64                                            1.43-6.el7                                                     base                                                39 k
 postgresql14                                                      x86_64                                            14.5-1PGDG.rhel7                                               pgdg14                                             1.5 M
 postgresql14-libs                                                 x86_64                                            14.5-1PGDG.rhel7                                               pgdg14                                             270 k
 postgresql14-server                                               x86_64                                            14.5-1PGDG.rhel7                                               pgdg14                                             5.5 M
 python3                                                           x86_64                                            3.6.8-18.el7                                                   updates                                             70 k
 python3-libs                                                      x86_64                                            3.6.8-18.el7                                                   updates                                            6.9 M
 python3-pip                                                       noarch                                            9.0.3-8.el7                                                    base                                               1.6 M
 python3-setuptools                                                noarch                                            39.2.0-10.el7                                                  base                                               629 k

Transaction Summary
=======================================================================================================================================================================================
Install  1 Package (+36 Dependent packages)

Total download size: 36 M
Installed size: 142 M
Is this ok [y/d/N]: y




инициализирую кластер

[root@postgres1989-03 /]# sudo /usr/pgsql-14/bin/postgresql-14-setup initdb
Initializing database ... OK


--------------------------------------------------------------------------------------------------------------



--------------------------------------------------------------------------------------------------------------
зайти вторым ssh (вторая сессия)

#зашел добавил сервер в автозагрузку

sudo systemctl enable --now postgresql-14
systemctl status postgresql-14
sudo su - postgres
--------------------------------------------------------------------------------------------------------------


--------------------------------------------------------------------------------------------------------------
запустить везде psql из под пользователя postgres
выполнено!



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