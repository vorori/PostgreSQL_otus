# 1)
создать ВМ с Ubuntu 20.04/22.04 или развернуть докер любым удобным способом

создал vm centos 7
добавил репозиторий

sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo


# 2)
поставить на нем Docker Engine

sudo yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin

sudo systemctl start docker

sudo systemctl enable docker

# 3)
сделать каталог /var/lib/postgres

mkdir /var/lib/postgres

# 4)
развернуть контейнер с PostgreSQL 14 смонтировав в него /var/lib/postgres


create network for postgre 

sudo docker network create my_network_postgre


sudo docker run -d --name server_postgres14 --network my_network_postgre -e POSTGRES_PASSWORD=postgres -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:14

docker container ls --all

# 5)
развернуть контейнер с клиентом postgres

sudo docker run -d -it --rm --network my_network_postgre --name pg-client postgres:14 psql -h server_postgres14 -U postgres 


# 6)
подключится из контейнера с клиентом к контейнеру с сервером и сделать таблицу с парой строк

sudo docker exec -it pg-client /bin/bash

psql -h server_postgres14 -U postgres

create table persons(id serial, first_name text, second_name text);
insert into persons(first_name, second_name) values('ivan', 'ivanov');
insert into persons(first_name, second_name) values('petr', 'petrov');

# 7)
подключится к контейнеру с сервером с ноутбука/компьютера извне инстансов GCP/ЯО/места установки докера

добавил в pg_hba.conf:
host    all             postgres        0.0.0.0/32              md5

применил наcтройки:

select pg_reload_conf();



установил на windows 10 клиент psql и подключился к моему каластеру postgres на виртуалке

C:\Users\Adm>psql -U postgres -h 158.160.5.207 -d postgres
Password for user postgres:
psql (15.0, server 14.5 (Debian 14.5-2.pgdg110+2))
WARNING: Console code page (866) differs from Windows code page (1251)
         8-bit characters might not work correctly. See psql reference
         page "Notes for Windows users" for details.
Type "help" for help.


postgres=# \dt
          List of relations
 Schema |  Name   | Type  |  Owner
--------+---------+-------+----------
 public | persons | table | postgres
(1 row)


postgres=# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
(2 rows)


# 8)
удалить контейнер с сервером

docker container ls --all

docker stop <Container_ID>

docker rm <Container_ID>


# 9)
создать его снова

sudo docker run -d --name server_postgres14 --network my_network_postgre -e POSTGRES_PASSWORD=postgres -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:14

sudo docker run -d -it --rm --network my_network_postgre --name pg-client postgres:14 psql -h server_postgres14 -U postgres 

# 10)
подключится снова из контейнера с клиентом к контейнеру с сервером

sudo docker exec -it pg-client /bin/bash
psql -h server_postgres14 -U postgres

# 11)
проверить, что данные остались на месте

данные остались на месте, поскольку мы смонтировали папку на локальном диске с помощью команды -v /var/lib/postgres:/var/lib/postgresql/data

postgres=# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
(2 rows)

# 12)
оставляйте в ЛК ДЗ комментарии что и как вы делали и как боролись с проблемами

очень понравилась статья на habr
https://habr.com/ru/post/310460/