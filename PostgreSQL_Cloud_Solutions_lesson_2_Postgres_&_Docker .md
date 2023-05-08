#### 1)

#### сделать в GCE/ЯО/Аналоги инстанс с Ubuntu 20.04


<pre>
создал vm centos 
</pre>

#### 2)

#### поставить на нем Docker Engine

<pre>
добавить официальный репозиторий Docker, загрузить последнюю версию Docker и установить ее
curl -fsSL https://get.docker.com/ | sh

sudo systemctl start docker
sudo systemctl status docker

</pre>

#### 3)

#### сделать каталог /var/lib/postgres

<pre>
mkdir /var/lib/postgres

</pre>

#### 4)

####  развернуть контейнер с PostgreSQL 14 смонтировав в него /var/lib/postgres

<pre>

создаю сеть для postgresql
sudo docker network create my_network_postgre

разворачиваем postgre
sudo docker run -d --name server_postgres14 --network my_network_postgre -e POSTGRES_PASSWORD=postgres -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:14

смотрим ставтус контенера
docker container ls --all

</pre>



#### 5)

####   развернуть контейнер с клиентом postgres

<pre>

sudo docker run -d -it --rm --network my_network_postgre --name pg-client postgres:14 psql -h server_postgres14 -U postgres

</pre>



#### 6)

####  подключится из контейнера с клиентом к контейнеру с сервером и сделать таблицу с парой строк

<pre>

подключаемся снач в контенер с клиентом
sudo docker exec -it pg-client /bin/bash

а теперь подключаемся из контенера с клиентом в контенер с БД
psql -h server_postgres14 -U postgres
или
получить ip адрес контенера с БД
docker inspect de43d1f0567f | grep "IPAddress"
psql -h 172.18.0.2 -U postgres


create table persons(id serial, first_name text, second_name text);
insert into persons(first_name, second_name) values('ivan7', 'ivanov7');
insert into persons(first_name, second_name) values('petr7', 'petrov7');

</pre>



#### 7)

####  подключится к контейнеру с сервером с ноутбука/компьютера извне инстансов GCP/ЯО/Аналоги

<pre>

добавил в pg_hba.conf: 
host all postgres 0.0.0.0/32 md5

применил наcтройки:
select pg_reload_conf();

установил на windows 10 клиент psql и подключился к моему каластеру postgres на виртуалке
psql -U postgres -h 158.160.13.204 -d postgres

</pre>



#### 8)

####  удалить контейнер с сервером

<pre>

docker container ls --all
docker stop <Container_ID>
docker rm <Container_ID>

</pre>


#### 9)

####  создать его заново

<pre>

sudo docker run -d --name server_postgres14 --network my_network_postgre -e POSTGRES_PASSWORD=postgres -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:14

</pre>



#### 10)

####   подключится снова из контейнера с клиентом к контейнеру с сервером

<pre>

подключаемся снач в контенер с клиентом
sudo docker exec -it pg-client /bin/bash

а теперь подключаемся из контенера с клиентом в контенер с БД
psql -h server_postgres14 -U postgres
или
получить ip адрес контенера с БД
docker inspect de43d1f0567f | grep "IPAddress"
psql -h 172.18.0.2 -U postgres

</pre>



#### 11)

####   проверить, что данные остались на месте

<pre>

данные на месте

postgres=# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan7      | ivanov7
  2 | petr7      | petrov7
(2 rows)


</pre>



#### 12)

####   оставляйте в ЛК ДЗ комментарии что и как вы делали и как боролись с проблемами

<pre>
боролся с лицензий от SmartGit установил более старую версию с бесплатной лицензией)

Начиная с версии 22.1 некоммерческая лицензия доступна только по запросу.
В блоге Syntevo от 19 октября 2022 г. объясняется, что варианты лицензирования были изменены в версии 22.1. 

Некоммерческая лицензия доступна только для:
разработчики с открытым исходным кодом,
студенты и сотрудники (избранных) учебных заведений, а также
благотворительные общественные организации.
У любого из них есть условия, которые необходимо выполнить , прежде чем может быть выдана некоммерческая лицензия.

SmartGit больше нельзя использовать для «хобби» , а это означает, что старые ответы неактуальны: по умолчанию теперь вы получаете копию SmartGit для оценки в течение 30 дней.


</pre>