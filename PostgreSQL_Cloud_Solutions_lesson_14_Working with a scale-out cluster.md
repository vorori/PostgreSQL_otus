
<pre>
---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
-------------------------------------------------------Подготовительные мероприятия----------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
</pre>




#### 1)

#### создеем vm-ки и устанвливаем настраиваем доп пакеты

<pre>

1.1)
создал vm 3 vm
2 cpu 3 озу  диск ssd
kub1 192.168.2.11
kub2 192.168.2.12
kub3 192.168.2.13

1.2)
подключаю Репозиторий EPEL 
sudo yum install epel-release

1.3)
выполняю настройку синхронизации времени на серверах:
yum install -y chrony
yum install -y chrony
yum install -y chrony

enable --now chronyd.service
enable --now chronyd.service
enable --now chronyd.service

1.4)
yum install -y glibc ncurses-libs tzdata
yum install -y glibc ncurses-libs tzdata
yum install -y glibc ncurses-libs tzdata
</pre>


#### 2)

#### скачиваем дистрибутивы cockroachdb на каждую VM и подготовительные мероприятия


<pre>

2.1)

wget https://binaries.cockroachdb.com/cockroach-v21.1.21.linux-amd64.tgz
wget https://binaries.cockroachdb.com/cockroach-v21.1.21.linux-amd64.tgz

--2023-07-03 21:07:40--  https://binaries.cockroachdb.com/cockroach-v21.1.21.linux-amd64.tgz
Resolving binaries.cockroachdb.com (binaries.cockroachdb.com)... 34.149.102.43
Connecting to binaries.cockroachdb.com (binaries.cockroachdb.com)|34.149.102.43|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 81628493 (78M) [binary/octet-stream]
Saving to: ‘cockroach-v21.1.21.linux-amd64.tgz’
100%[======================================================================================>] 81,628,493  6.50MB/s   in 33s
2023-07-03 21:08:14 (2.39 MB/s) - ‘cockroach-v21.1.21.linux-amd64.tgz’ saved [81628493/81628493]


2.2)

подготовительные мероприятия для cockroachdb
извлекаем загруженный архив 
tar xf cockroach-v21.1.21.linux-amd64.tgz

2.3)

создаем каталоги для программного обеспечения CockroachDB и связанных файлов.
mkdir -p /opt/cockroachdb/{bin,certs,private}
mkdir -p /opt/cockroachdb/{bin,certs,private}
mkdir -p /opt/cockroachdb/{bin,certs,private}

2.4)

копируем извлеченные файлы в каталог /opt/cockroachdb/bin .
cp -i cockroach-v21.1.21.linux-amd64/cockroach /opt/cockroachdb/bin/

2.5)
проверяем права доступа к файлу таракана .
ls -al /opt/cockroachdb/bin/
total 180568
drwxr-xr-x. 2 cockroach cockroach        23 Jul  3 21:18 .
drwxr-xr-x. 5 cockroach cockroach        45 Jul  3 21:13 ..
-rwxr-xr-x. 1 root      root      184898616 Jul  3 21:18 cockroach

2.6)
создаем пользователя чтобы владеть программным обеспечением и процессами CockroachDB. 
Также предоставьте право собственности на файлы этому пользователю.
useradd -r cockroach
chown -R cockroach.cockroach /opt/cockroachdb/

2.7)
создаем ссылку на файл тараканов в каталоге /usr/local/bin/
ln -s /opt/cockroachdb/bin/cockroach /usr/local/bin/cockroach

2.8)
проверяю версию
cockroach version
Build Tag:        v21.1.21
Build Time:       2022/09/15 18:11:02
Distribution:     CCL
Platform:         linux amd64 (x86_64-unknown-linux-gnu)
Go Version:       go1.15.14
C Compiler:       gcc 6.5.0
Build Commit ID:  dd345173f3ef06b031cf470c67ae14dd0503a091
Build Type:       release
</pre>



#### 3)
#### Примечание

<pre>
необходимо выполнить на каждом узле кластера CockroachDB. 
Принимая во внимание, что последующие шаги относятся к узлам и должны выполняться только на упомянутых узлах.

Кластер CockroachDB можно настроить в безопасном и небезопасном режимах.

Конфигурация кластера CockroachDB в небезопасном режиме довольно проста, но не требует принудительного
шифрования межкластерной связи.

Принимая во внимание, что безопасный режим использует сертификаты SSL/TLS для принудительного 
шифрования межкластерной связи и авторизации.

необходимо создать центр сертификации (ЦС), который будет использоваться 
для цифровой подписи любого сертификата, который вы создадите для своего безопасного кластера CockroachDB.
</pre>



#### 4) 

#### работа с сертификатами


<pre>

4.1)
создаем центр сертификации
cockroach cert create-ca \
--certs-dir=/opt/cockroachdb/certs \
--ca-key=/opt/cockroachdb/private/ca.key

4.2)
создаем сертификат SSL/TLS для нашего первого узла CockroachDB (kub1) с помощью следующей команды.
cockroach cert create-node \
192.168.2.11 \
kub1 \
localhost \
--certs-dir=/opt/cockroachdb/certs \
--ca-key=/opt/cockroachdb/private/ca.key


4.3)
создаем сертификат SSL/TLS для клиента CockroachDB, выполнив следующую команду.
cockroach cert create-client \
root \
--certs-dir=/opt/cockroachdb/certs \
--ca-key=/opt/cockroachdb/private/ca.key


4.4)
копирую сертификаты SSL/TLS на другие узлы кластера баз данных.

scp /opt/cockroachdb/certs/* \
root@kub2:/opt/cockroachdb/certs/

scp /opt/cockroachdb/certs/* \
root@kub3:/opt/cockroachdb/certs/


4.5)
копирую ключ центра сертификации на другие узлы, чтобы мы могли создать SSL/TLS на этих узлах.
scp /opt/cockroachdb/private/* root@kub2:/opt/cockroachdb/private/
scp /opt/cockroachdb/private/* root@kub3:/opt/cockroachdb/private/


4.6)
подключаюсь к kub2 и  kub3 
удаляю сертификат/ключ узла, который мы сгенерировали на узле kub1
rm -f /opt/cockroachdb/certs/node.*


создаем сертификат SSL/TLS для узла kub2 и  kub3 следующим образом.
cockroach cert create-node \
192.168.2.12 \
kub2 \
localhost \
--certs-dir=/opt/cockroachdb/certs \
--ca-key=/opt/cockroachdb/private/ca.key


cockroach cert create-node \
192.168.2.13 \
kub3 \
localhost \
--certs-dir=/opt/cockroachdb/certs \
--ca-key=/opt/cockroachdb/private/ca.key

4.7)
ОБЯЗАТЕЛЬНО ПРАВА НА СЕРТИФИКАТЫ!!!!!
chown cockroach:cockroach /opt/cockroachdb/certs/ -R
chown cockroach:cockroach /opt/cockroachdb/certs/ -R
chown cockroach:cockroach /opt/cockroachdb/certs/ -R
</pre>



#### 5)

#### создаем сервисную единицу Systemd

<pre>

5.1)

Чтобы включить автозапуск сервера CockroachDB во время запуска Linux, вам необходимо создать сервисную единицу systemd.
создаю файл сервисной единицы systemd с помощью редактора vim
ВАЖНО!!!!!!!!!!!!!!
НЕОБХОДИМО заменить --advertise-addr на имя хоста узла CockroachDB, на создаю эту службу systemd.

5.2)
vim /etc/systemd/system/cockroachdb.service

---------------------------------------------------------------
[Unit]
Description=Cockroach Database cluster node
Requires=network.target

[Service]
Type=notify
WorkingDirectory=/opt/cockroachdb
ExecStart=/usr/local/bin/cockroach start --certs-dir=/opt/cockroachdb/certs --advertise-addr=kub1 --join=kub1,kub2,kub3
TimeoutStopSec=60
Restart=always
RestartSec=10
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=cockroach
User=cockroach

[Install]
WantedBy=default.target
---------------------------------------------------------------
</pre>




#### 6)

#### запускаем кластер

<pre>

6.1)
преред запуском настраиваем брандмауэр Linux:
открываем порты
firewall-cmd --permanent --add-port={8080,26257}/tcp 
firewall-cmd --reload 

где:
Сервисные порты CockroachDB по умолчанию: 8080/tcp для пользовательского интерфейса веб-администратора, 26257/tcp для интерфейса SQL.

6.2)
добавляем в автозапуск и проверяем
systemctl enable --now cockroachdb.service
systemctl status cockroachdb.service
chown cockroach:cockroach /opt/cockroachdb/certs/ -R


[root@kub1 ~]# systemctl status cockroachdb.service
● cockroachdb.service - Cockroach Database cluster node
   Loaded: loaded (/etc/systemd/system/cockroachdb.service; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2023-07-03 22:15:50 MSK; 1min 29s ago
 Main PID: 1833 (cockroach)
   CGroup: /system.slice/cockroachdb.service
           └─1833 /usr/local/bin/cockroach start --certs-dir=/opt/cockroachdb/certs --advertise-addr=kub1 --join=kub1,kub2,kub3

Jul 03 22:15:50 kub1 systemd[1]: Started Cockroach Database cluster node.
Jul 03 22:16:20 kub1 cockroach[1833]: *
Jul 03 22:16:20 kub1 cockroach[1833]: * WARNING: The server appears to be unable to contact the other nodes in the cluster. Please try:
Jul 03 22:16:20 kub1 cockroach[1833]: *
Jul 03 22:16:20 kub1 cockroach[1833]: * - starting the other nodes, if you haven't already;
Jul 03 22:16:20 kub1 cockroach[1833]: * - double-checking that the '--join' and '--listen'/'--advertise' flags are set up correctly;
Jul 03 22:16:20 kub1 cockroach[1833]: * - running the 'cockroach init' command if you are trying to initialize a new cluster.
Jul 03 22:16:20 kub1 cockroach[1833]: *
Jul 03 22:16:20 kub1 cockroach[1833]: * If problems persist, please see https://www.cockroachlabs.com/docs/v21.1/cluster-setup-troubleshooting.html.
Jul 03 22:16:20 kub1 cockroach[1833]: *


6.3)
проверяю порты службы CockroachDB, чтобы убедиться, что служба CockroachDB запускается без ошибок.
ss -tulpn | grep cockroach
tcp    LISTEN     0      128    [::]:8080               [::]:*                   users:(("cockroach",pid=1809,fd=20))
tcp    LISTEN     0      128    [::]:26257              [::]:*                   users:(("cockroach",pid=1809,fd=21))

6.4)
Инициализировать кластер CockroachDB
на любом узле CockroachDB выполняем для инициализации кластера.
cockroach init --certs-dir=/opt/cockroachdb/certs --host=kub1:26257

6.5)
cockroach init --certs-dir=/opt/cockroachdb/certs --host=kub1:26257
Cluster successfully initialized
</pre>


#### 7)

#### проверка работоспособности

<pre>

7.1)
Смотрим статус
cockroach node status --certs-dir=/opt/cockroachdb/certs
  id |  address   | sql_address |  build   |         started_at         |         updated_at         | locality | is_available | is_live
-----+------------+-------------+----------+----------------------------+----------------------------+----------+--------------+----------
   1 | kub1:26257 | kub1:26257  | v21.1.21 | 2023-07-03 19:27:50.492607 | 2023-07-04 18:18:30.540129 |          | true         | true
   2 | kub2:26257 | kub2:26257  | v21.1.21 | 2023-07-03 19:27:01.207839 | 2023-07-04 18:18:30.746901 |          | true         | true
   3 | kub3:26257 | kub3:26257  | v21.1.21 | 2023-07-03 19:26:54.413958 | 2023-07-04 18:18:32.963394 |          | true         | true
(3 rows)


7.2)
подключаемся
cockroach sql --certs-dir=/opt/cockroachdb/certs --host=kub1:26257
cockroach sql --certs-dir=/opt/cockroachdb/certs --host=kub2:26257
cockroach sql --certs-dir=/opt/cockroachdb/certs --host=kub3:26257


cockroach sql --certs-dir=/opt/cockroachdb/certs --host=kub1:26257
#
# Welcome to the CockroachDB SQL shell.
# All statements must be terminated by a semicolon.
# To exit, type: \q.
#
# Server version: CockroachDB CCL v21.1.21 (x86_64-unknown-linux-gnu, built 2022/09/15 18:11:02, go1.15.14) (same version as client)
# Cluster ID: 70864dc3-ee6f-499d-8a8c-e8ba70417fdb
#
# Enter \? for a brief introduction.
#
root@kub1:26257/defaultdb> SHOW DATABASES;
  database_name | owner | primary_region | regions | survival_goal
----------------+-------+----------------+---------+----------------
  defaultdb     | root  | NULL           | {}      | NULL
  postgres      | root  | NULL           | {}      | NULL
  system        | node  | NULL           | {}      | NULL
(3 rows)

Time: 1ms total (execution 1ms / network 0ms)

root@kub1:26257/defaultdb>
</pre>



<pre>
---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
-------------------------------------------------------Потесировать dataset с чикагскими такси-----------------------------------------
-------------------------------------------------------Загружаем DataSet для ДЗ размером 10 Гб-----------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
</pre>


### 8)

#### connect

<pre>
8.1)
cockroach sql --certs-dir=/opt/cockroachdb/certs --host=kub1:26257 --database=test
cockroach sql --certs-dir=/opt/cockroachdb/certs --host=kub1:26257 --database=test
cockroach sql --certs-dir=/opt/cockroachdb/certs --host=kub1:26257 --database=test
</pre>


#### Создаем базу

<pre>

8.2)
cockroach sql --certs-dir=/opt/cockroachdb/certs --host=kub1:26257
SHOW DATABASES;
\l

root@kub1:26257/defaultdb> create database test;
CREATE DATABASE

Time: 62ms total (execution 60ms / network 2ms)

root@kub1:26257/defaultdb> \l
  database_name | owner | primary_region | regions | survival_goal
----------------+-------+----------------+---------+----------------
  defaultdb     | root  | NULL           | {}      | NULL
  postgres      | root  | NULL           | {}      | NULL
  system        | node  | NULL           | {}      | NULL
  test          | root  | NULL           | {}      | NULL
(4 rows)

Time: 2ms total (execution 2ms / network 0ms)


root@kub1:26257/defaultdb> use test;
SET

Time: 11ms total (execution 11ms / network 0ms)
root@kub1:26257/test>
</pre>



#### подготовка к загрузке

<pre>
8.3)

#### документация
https://www.cockroachlabs.com/docs/v21.1/use-userfile-for-bulk-operations
https://www.cockroachlabs.com/docs/v21.1/use-userfile-for-bulk-operations
https://www.cockroachlabs.com/docs/v21.1/use-userfile-for-bulk-operations

drop table taxi_trips;
drop table taxi_trips;
drop table taxi_trips;

create table taxi_trips (
unique_key text, 
taxi_id text, 
trip_start_timestamp TIMESTAMP, 
trip_end_timestamp TIMESTAMP, 
trip_seconds bigint, 
trip_miles numeric, 
pickup_census_tract bigint, 
dropoff_census_tract bigint, 
pickup_community_area bigint, 
dropoff_community_area bigint, 
fare numeric, 
tips numeric, 
tolls numeric, 
extras numeric, 
trip_total numeric, 
payment_type text, 
company text, 
pickup_latitude numeric, 
pickup_longitude numeric, 
pickup_location text, 
dropoff_latitude numeric, 
dropoff_longitude numeric, 
dropoff_location text
);

</pre>



#### выполняем загрузку данных

<pre>

8.4)
 
cockroach userfile upload /tmp/taxi/taxi.csv.000000000000 /taxi.csv.000000000000 --certs-dir=/opt/cockroachdb/certs
successfully uploaded to userfile://defaultdb.public.userfiles_root/taxi.csv.000000000000

[root@kub1 ~]# cd /tmp/taxi
[root@kub1 taxi]# ls -l
total 6782408
-rwxrwxrwx. 1 root root 267363203 Jul  5 14:23 taxi.csv.000000000000
-rwxrwxrwx. 1 root root 267344607 Jun 15 18:01 taxi.csv.000000000001
-rwxrwxrwx. 1 root root 267390243 Jul  4 15:06 taxi.csv.000000000002
-rwxrwxrwx. 1 root root 267417696 Jun 15 17:50 taxi.csv.000000000003
-rwxrwxrwx. 1 root root 266371749 Jun 15 18:02 taxi.csv.000000000004
-rwxrwxrwx. 1 root root 266233035 Jun 15 18:03 taxi.csv.000000000005
-rwxrwxrwx. 1 root root 266219968 Jun 15 18:03 taxi.csv.000000000006
-rwxrwxrwx. 1 root root 267380379 Jun 15 16:35 taxi.csv.000000000007
-rwxrwxrwx. 1 root root 267367985 Jun 15 17:53 taxi.csv.000000000008
-rwxrwxrwx. 1 root root 267278191 Jun 15 17:53 taxi.csv.000000000009
-rwxrwxrwx. 1 root root 267389474 Jun 15 17:54 taxi.csv.000000000010
-rwxrwxrwx. 1 root root 267331103 Jun 15 18:04 taxi.csv.000000000011
-rwxrwxrwx. 1 root root 266153465 Jun 15 17:55 taxi.csv.000000000012
-rwxrwxrwx. 1 root root 267368711 Jun 15 16:38 taxi.csv.000000000013
-rwxrwxrwx. 1 root root 265622273 Jun 15 17:55 taxi.csv.000000000014
-rwxrwxrwx. 1 root root 267319434 Jun 15 18:04 taxi.csv.000000000015
-rwxrwxrwx. 1 root root 267311996 Jun 15 18:04 taxi.csv.000000000016
-rwxrwxrwx. 1 root root 267325169 Jun 15 17:56 taxi.csv.000000000017
-rwxrwxrwx. 1 root root 267373075 Jun 15 16:38 taxi.csv.000000000018
-rwxrwxrwx. 1 root root 267377563 Jun 15 18:04 taxi.csv.000000000019
-rwxrwxrwx. 1 root root 267374748 Jun 15 17:56 taxi.csv.000000000020
-rwxrwxrwx. 1 root root 267391164 Jun 15 17:56 taxi.csv.000000000021
-rwxrwxrwx. 1 root root 267335900 Jun 15 18:04 taxi.csv.000000000022
-rwxrwxrwx. 1 root root 267348008 Jun 15 18:04 taxi.csv.000000000023
-rwxrwxrwx. 1 root root 267374332 Jun 15 18:04 taxi.csv.000000000024
-rwxrwxrwx. 1 root root 267369255 Jun 15 18:04 taxi.csv.000000000025


8.5)

#Загрузить весь список наших файлов действие 1
cockroach userfile list '*.csv.*' --certs-dir=/opt/cockroachdb/certs
cockroach userfile list '*.csv.*' --certs-dir=/opt/cockroachdb/certs
cockroach userfile list '*.csv.*' --certs-dir=/opt/cockroachdb/certs

8.6)

#Загрузить весь список наших файлов действие 2
cockroach userfile upload /tmp/taxi/taxi.csv.000000000000 /taxi.csv.000000000000 --certs-dir=/opt/cockroachdb/certs
cockroach userfile upload /tmp/taxi/taxi.csv.000000000001 /taxi.csv.000000000001 --certs-dir=/opt/cockroachdb/certs
cockroach userfile upload /tmp/taxi/taxi.csv.000000000002 /taxi.csv.000000000002 --certs-dir=/opt/cockroachdb/certs
cockroach userfile upload /tmp/taxi/taxi.csv.000000000003 /taxi.csv.000000000003 --certs-dir=/opt/cockroachdb/certs
cockroach userfile upload /tmp/taxi/taxi.csv.000000000004 /taxi.csv.000000000004 --certs-dir=/opt/cockroachdb/certs
cockroach userfile upload /tmp/taxi/taxi.csv.000000000005 /taxi.csv.000000000005 --certs-dir=/opt/cockroachdb/certs
cockroach userfile upload /tmp/taxi/taxi.csv.000000000006 /taxi.csv.000000000006 --certs-dir=/opt/cockroachdb/certs
cockroach userfile upload /tmp/taxi/taxi.csv.000000000007 /taxi.csv.000000000007 --certs-dir=/opt/cockroachdb/certs
cockroach userfile upload /tmp/taxi/taxi.csv.000000000008 /taxi.csv.000000000008 --certs-dir=/opt/cockroachdb/certs
cockroach userfile upload /tmp/taxi/taxi.csv.000000000009 /taxi.csv.000000000009 --certs-dir=/opt/cockroachdb/certs
cockroach userfile upload /tmp/taxi/taxi.csv.000000000010 /taxi.csv.000000000010 --certs-dir=/opt/cockroachdb/certs
cockroach userfile upload /tmp/taxi/taxi.csv.000000000011 /taxi.csv.000000000011 --certs-dir=/opt/cockroachdb/certs
cockroach userfile upload /tmp/taxi/taxi.csv.000000000012 /taxi.csv.000000000012 --certs-dir=/opt/cockroachdb/certs
cockroach userfile upload /tmp/taxi/taxi.csv.000000000013 /taxi.csv.000000000013 --certs-dir=/opt/cockroachdb/certs
cockroach userfile upload /tmp/taxi/taxi.csv.000000000014 /taxi.csv.000000000014 --certs-dir=/opt/cockroachdb/certs
cockroach userfile upload /tmp/taxi/taxi.csv.000000000015 /taxi.csv.000000000015 --certs-dir=/opt/cockroachdb/certs
cockroach userfile upload /tmp/taxi/taxi.csv.000000000016 /taxi.csv.000000000016 --certs-dir=/opt/cockroachdb/certs
cockroach userfile upload /tmp/taxi/taxi.csv.000000000017 /taxi.csv.000000000017 --certs-dir=/opt/cockroachdb/certs
cockroach userfile upload /tmp/taxi/taxi.csv.000000000018 /taxi.csv.000000000018 --certs-dir=/opt/cockroachdb/certs
cockroach userfile upload /tmp/taxi/taxi.csv.000000000019 /taxi.csv.000000000019 --certs-dir=/opt/cockroachdb/certs
cockroach userfile upload /tmp/taxi/taxi.csv.000000000020 /taxi.csv.000000000020 --certs-dir=/opt/cockroachdb/certs
cockroach userfile upload /tmp/taxi/taxi.csv.000000000021 /taxi.csv.000000000021 --certs-dir=/opt/cockroachdb/certs
cockroach userfile upload /tmp/taxi/taxi.csv.000000000022 /taxi.csv.000000000022 --certs-dir=/opt/cockroachdb/certs
cockroach userfile upload /tmp/taxi/taxi.csv.000000000023 /taxi.csv.000000000023 --certs-dir=/opt/cockroachdb/certs
cockroach userfile upload /tmp/taxi/taxi.csv.000000000024 /taxi.csv.000000000024 --certs-dir=/opt/cockroachdb/certs
cockroach userfile upload /tmp/taxi/taxi.csv.000000000025 /taxi.csv.000000000025 --certs-dir=/opt/cockroachdb/certs


или

установить клиент psql 
CREATE USER vorori WITH LOGIN PASSWORD 'my_pass';
altet user vorori WITH CREATEDB;
psql -p 26257 -h 192.168.2.11 -U vorori -d test
GRANT ALL ON DATABASE test TO vorori;

test=> \COPY taxi_trips2 FROM '/tmp/taxi/taxi.csv.000000000000' DELIMITER ',' CSV;
test=> \COPY taxi_trips2 FROM '/tmp/taxi/taxi.csv.000000000001' DELIMITER ',' CSV;
test=> \COPY taxi_trips2 FROM '/tmp/taxi/taxi.csv.000000000002' DELIMITER ',' CSV;
test=> \COPY taxi_trips2 FROM '/tmp/taxi/taxi.csv.000000000003' DELIMITER ',' CSV;
и т. д.

8.7)

#если надо удалить delete
cockroach userfile delete /taxi.csv.0000000000** --certs-dir=/opt/cockroachdb/certs
cockroach userfile delete /taxi.csv.0000000000** --certs-dir=/opt/cockroachdb/certs
cockroach userfile delete /taxi.csv.0000000000** --certs-dir=/opt/cockroachdb/certs

8.8)

cockroach sql --certs-dir=/opt/cockroachdb/certs --host=kub1:26257 --database=test
cockroach sql --certs-dir=/opt/cockroachdb/certs --host=kub1:26257 --database=test
cockroach sql --certs-dir=/opt/cockroachdb/certs --host=kub1:26257 --database=test


#пример для загрузки всех dataset taxi
import into taxi_trips (unique_key,
taxi_id,
trip_start_timestamp,
trip_end_timestamp,
trip_seconds,
trip_miles,
pickup_census_tract,
dropoff_census_tract,
pickup_community_area,
dropoff_community_area,
fare,
tips,
tolls,extras,
trip_total,
payment_type,
company,
pickup_latitude,
pickup_longitude,
pickup_location,
dropoff_latitude,
dropoff_longitude,
dropoff_location) 
csv data ('userfile:///taxi.csv.000000000***') with skip = '1', nullif = '';

#пример для загрузки одного dataset taxi
import into taxi_trips (unique_key,
taxi_id,
trip_start_timestamp,
trip_end_timestamp,
trip_seconds,
trip_miles,
pickup_census_tract,
dropoff_census_tract,
pickup_community_area,
dropoff_community_area,
fare,
tips,
tolls,extras,
trip_total,
payment_type,
company,
pickup_latitude,
pickup_longitude,
pickup_location,
dropoff_latitude,
dropoff_longitude,
dropoff_location) 
csv data ('userfile:///taxi.csv.000000000002') with skip = '1', nullif = '';





#проверил количество записей в таблице всё ок - данные загружены нормально
select count(*) from taxi_trips;
   count
------------
  16929630
(1 row)

</pre>



### 9)

#### создал и установил postgres 15 подтюнил оснровные настройки PGTune под озу 4 gb CPU 2 диск ssd

<pre>
sudo yum -y install epel-release
sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
sudo yum -y repolist
sudo yum -y install postgresql15-contrib

sudo /usr/pgsql-15/bin/postgresql-15-setup initdb
systemctl enable postgresql-15.service
systemctl status postgresql-15.service
systemctl restart postgresql-15.service

#создал табличку влил данные
create database test;

create table taxi_trips (
unique_key text, 
taxi_id text, 
trip_start_timestamp TIMESTAMP, 
trip_end_timestamp TIMESTAMP, 
trip_seconds bigint, 
trip_miles numeric, 
pickup_census_tract bigint, 
dropoff_census_tract bigint, 
pickup_community_area bigint, 
dropoff_community_area bigint, 
fare numeric, 
tips numeric, 
tolls numeric, 
extras numeric, 
trip_total numeric, 
payment_type text, 
company text, 
pickup_latitude numeric, 
pickup_longitude numeric, 
pickup_location text, 
dropoff_latitude numeric, 
dropoff_longitude numeric, 
dropoff_location text
);

#влил тот же объем данных 
for f in *.csv*; do psql -U postgres -d test -c "\\COPY taxi_trips FROM PROGRAM 'cat $f' CSV HEADER"; done
for f in *.csv*; do psql -U postgres -d test -c "\\COPY taxi_trips FROM PROGRAM 'cat $f' CSV HEADER"; done
for f in *.csv*; do psql -U postgres -d test -c "\\COPY taxi_trips FROM PROGRAM 'cat $f' CSV HEADER"; done
</pre>





### Выполнил тестовые запросы

<pre>
---------------------------------------------------------------------------------------------------------------------------------------
запросы:
------
SELECT payment_type, round(sum(tips)/sum(trip_total)*100, 0) + 0 as tips_percent, count(*) as c 
FROM taxi_trips
group by payment_type
order by c;
------

------
select count(*) from taxi_trips;
------

------
select count(unique_key) from taxi_trips;
create index unique_key_idx on taxi_trips(unique_key);
------

------
create table t1 as select generate_series(1,1000000) as colA;
------

------
select sum(colA) from t1;
------
---------------------------------------------------------------------------------------------------------------------------------------
</pre>


#### ИТОГО Сравнение производительности запросов к БД в кластере CockroachDB и БД на одном инстансе PostgreSQL

<pre>
                                                                             postgres            CockroachDB   
время выполнение основного запроса для taxi_trips                            02 min 01.60 sec    51.257 sec
время запроса count(*) талица taxi_trips                                     03 min 20 sec       20.7  sec
время запроса count(unique_key) талица taxi_trips с индексом(unique_key)     7.37  sec           7.06  sec																			 
тестовая генерация данных для таблицы t1                                     2.850 sec           2.840 sec
посчитали сумму sum(colA) для таблицы t1                                     1.372 sec           395 ms


Итог: тестирование на БД в кластере CockroachDB производилось "из коробки", без дополнительных настроек производительности, 
и результаты довольно неплохие;
</pre>



<pre>
---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
-----------------------------------------------Описать что и как делали и с какими проблемами столкнулись------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
</pre>


<pre>
траблошутил служба cockroach не стартовала столкнулся c неверными правами доступа на сертификатах
следом вылезла проблема с параметром  AllowZoneDrifting в  конфиге /etc/firewalld/firewalld.conf 
cockroach не запустилась пока не поменял параметр с yes на no.
</pre>