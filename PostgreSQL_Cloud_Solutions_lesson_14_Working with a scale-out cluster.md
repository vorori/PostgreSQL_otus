https://www.centlinux.com/2020/11/install-cockroachdb-secure-cluster-on-centos-8.html
https://www.centlinux.com/2020/11/install-cockroachdb-secure-cluster-on-centos-8.html
https://www.centlinux.com/2020/11/install-cockroachdb-secure-cluster-on-centos-8.html


создал vm 3 vm
2 cpu 4 озу  диск ssd
kub1 192.168.2.11
kub2 192.168.2.12
kub3 192.168.2.13


Репозиторий EPEL 
sudo yum install epel-release
sudo yum install epel-release
sudo yum install epel-release


Настройка синхронизации времени на сервере Linux:
yum install -y chrony
yum install -y chrony
yum install -y chrony

enable --now chronyd.service
enable --now chronyd.service
enable --now chronyd.service

yum install -y glibc ncurses-libs tzdata
yum install -y glibc ncurses-libs tzdata
yum install -y glibc ncurses-libs tzdata


скачиваем cockroachdb
wget https://binaries.cockroachdb.com/cockroach-v21.1.21.linux-amd64.tgz
[root@kub3 tmp]# wget https://binaries.cockroachdb.com/cockroach-v21.1.21.linux-amd64.tgz
--2023-07-03 21:07:40--  https://binaries.cockroachdb.com/cockroach-v21.1.21.linux-amd64.tgz
Resolving binaries.cockroachdb.com (binaries.cockroachdb.com)... 34.149.102.43
Connecting to binaries.cockroachdb.com (binaries.cockroachdb.com)|34.149.102.43|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 81628493 (78M) [binary/octet-stream]
Saving to: ‘cockroach-v21.1.21.linux-amd64.tgz’

100%[=======================================================================================================================================================>] 81,628,493  6.50MB/s   in 33s

2023-07-03 21:08:14 (2.39 MB/s) - ‘cockroach-v21.1.21.linux-amd64.tgz’ saved [81628493/81628493]


подготовительные мероприятия
извлекаем загруженный архив 
tar xf cockroach-v21.1.21.linux-amd64.tgz

Создайте каталоги для программного обеспечения CockroachDB и связанных файлов.
mkdir -p /opt/cockroachdb/{bin,certs,private}
mkdir -p /opt/cockroachdb/{bin,certs,private}
mkdir -p /opt/cockroachdb/{bin,certs,private}

копируем извлеченные файлы в каталог /opt/cockroachdb/bin .
cp -i cockroach-v21.1.21.linux-amd64/cockroach /opt/cockroachdb/bin/

проверяем права доступа к файлу таракана .
[root@kub1 tmp]# ls -al /opt/cockroachdb/bin/
total 180568
drwxr-xr-x. 2 cockroach cockroach        23 Jul  3 21:18 .
drwxr-xr-x. 5 cockroach cockroach        45 Jul  3 21:13 ..
-rwxr-xr-x. 1 root      root      184898616 Jul  3 21:18 cockroach

создаем пользователя  чтобы владеть программным обеспечением и процессами CockroachDB. 
Также предоставьте право собственности на файлы этому пользователю.
useradd -r cockroach
chown -R cockroach.cockroach /opt/cockroachdb/

сощдаем ссылку на файл тараканов в каталоге /usr/local/bin/
ln -s /opt/cockroachdb/bin/cockroach /usr/local/bin/cockroach

проверяю версию
[root@kub1 tmp]# cockroach version
Build Tag:        v21.1.21
Build Time:       2022/09/15 18:11:02
Distribution:     CCL
Platform:         linux amd64 (x86_64-unknown-linux-gnu)
Go Version:       go1.15.14
C Compiler:       gcc 6.5.0
Build Commit ID:  dd345173f3ef06b031cf470c67ae14dd0503a091
Build Type:       release



Примечание:
необходимо выполнить на каждом узле кластера CockroachDB. 
Принимая во внимание, что последующие шаги относятся к узлам и должны выполняться только на упомянутых узлах.

Кластер CockroachDB можно настроить в безопасном и небезопасном режимах.

Конфигурация кластера CockroachDB в небезопасном режиме довольно проста, но не требует принудительного
шифрования межкластерной связи.

Принимая во внимание, что безопасный режим использует сертификаты SSL/TLS для принудительного 
шифрования межкластерной связи и авторизации.

необходимо создать центр сертификации (ЦС), который будет использоваться 
для цифровой подписи любого сертификата, который вы создадите для своего безопасного кластера CockroachDB.


создаем центр сертификации
cockroach cert create-ca \
--certs-dir=/opt/cockroachdb/certs \
--ca-key=/opt/cockroachdb/private/ca.key



создаем сертификат SSL/TLS для нашего первого узла CockroachDB (kub1) с помощью следующей команды.
cockroach cert create-node \
192.168.2.11 \
kub1 \
localhost \
--certs-dir=/opt/cockroachdb/certs \
--ca-key=/opt/cockroachdb/private/ca.key



создаем сертификат SSL/TLS для клиента CockroachDB, выполнив следующую команду.
cockroach cert create-client \
root \
--certs-dir=/opt/cockroachdb/certs \
--ca-key=/opt/cockroachdb/private/ca.key


копирую сертификаты SSL/TLS на другие узлы кластера баз данных.
scp /opt/cockroachdb/certs/* \
root@kub2:/opt/cockroachdb/certs/

scp /opt/cockroachdb/certs/* \
root@kub3:/opt/cockroachdb/certs/

копирую ключ центра сертификации на другие узлы, чтобы мы могли создать SSL/TLS на этих узлах.
scp /opt/cockroachdb/private/* root@kub2:/opt/cockroachdb/private/
scp /opt/cockroachdb/private/* root@kub3:/opt/cockroachdb/private/


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



создаем сервисную единицу Systemd:
Чтобы включить автозапуск сервера CockroachDB во время запуска Linux, вам необходимо создать сервисную единицу systemd.
создаю файл сервисной единицы systemd с помощью редактора vim
ВАЖНО!!!!!!!!!!!!!!
НЕОБХОДИМОзаменить --advertise-addr на имя хоста узла CockroachDB, на создаю эту службу systemd.

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


ОБЯЗАТЕЛЬНО ПРАВА НА СЕРТИФИКАТЫ!!!!!
chown cockroach:cockroach /opt/cockroachdb/certs/ -R
chown cockroach:cockroach /opt/cockroachdb/certs/ -R
chown cockroach:cockroach /opt/cockroachdb/certs/ -R


настраиваем брандмауэр Linux:
открываем порты
firewall-cmd --permanent --add-port={8080,26257}/tcp 
firewall-cmd --reload 

где:
Сервисные порты CockroachDB по умолчанию: 8080/tcp для пользовательского интерфейса веб-администратора, 26257/tcp для интерфейса SQL.


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



проверяю порты службы CockroachDB, чтобы убедиться, что служба CockroachDB запускается без ошибок.
ss -tulpn | grep cockroach
tcp    LISTEN     0      128    [::]:8080               [::]:*                   users:(("cockroach",pid=1809,fd=20))
tcp    LISTEN     0      128    [::]:26257              [::]:*                   users:(("cockroach",pid=1809,fd=21))


Инициализировать кластер CockroachDB
на любом узле CockroachDB выполняем для инициализации кластера.
cockroach init --certs-dir=/opt/cockroachdb/certs --host=kub1:26257


cockroach init --certs-dir=/opt/cockroachdb/certs --host=kub1:26257
Cluster successfully initialized


Смотрим статус
cockroach node status --certs-dir=/opt/cockroachdb/certs
  id |  address   | sql_address |  build   |         started_at         |         updated_at         | locality | is_available | is_live
-----+------------+-------------+----------+----------------------------+----------------------------+----------+--------------+----------
   1 | kub1:26257 | kub1:26257  | v21.1.21 | 2023-07-03 19:27:50.492607 | 2023-07-04 18:18:30.540129 |          | true         | true
   2 | kub2:26257 | kub2:26257  | v21.1.21 | 2023-07-03 19:27:01.207839 | 2023-07-04 18:18:30.746901 |          | true         | true
   3 | kub3:26257 | kub3:26257  | v21.1.21 | 2023-07-03 19:26:54.413958 | 2023-07-04 18:18:32.963394 |          | true         | true
(3 rows)


Подключитесь к оболочке SQL  с помощью следующей команды
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


<pre>
---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
-------------------------------------------------------Потесировать dataset с чикагскими такси-----------------------------------------
-------------------------------------------------------Загружаем DataSet для ДЗ размером 10 Гб-----------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
</pre>


Создаем базу
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






<pre>
---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------кластер на postgres 15 -------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
</pre>


sudo yum -y install epel-release
sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
sudo yum -y repolist
sudo yum -y install postgresql15-contrib

sudo /usr/pgsql-15/bin/postgresql-15-setup initdb
systemctl enable postgresql-15.service
systemctl status postgresql-15.service
systemctl restart postgresql-15.service


#### создал и установил с подтюнил postgres 15
<pre>
3.1) 

создал и установил с подтюнил postgres 15 озу 4 gb CPU 2 диск ssd
влил тот же объем данных что и на кластер gp

ALTER SYSTEM SET
 max_connections = '40';
ALTER SYSTEM SET
 shared_buffers = '1536MB';
ALTER SYSTEM SET
 effective_cache_size = '4608MB';
ALTER SYSTEM SET
 maintenance_work_mem = '500MB';
ALTER SYSTEM SET
 checkpoint_completion_target = '0.9';
ALTER SYSTEM SET
 wal_buffers = '16MB';
ALTER SYSTEM SET
 default_statistics_target = '100';
ALTER SYSTEM SET
 random_page_cost = '4';
ALTER SYSTEM SET
 effective_io_concurrency = '2';
ALTER SYSTEM SET
 work_mem = '19660kB';
ALTER SYSTEM SET
 min_wal_size = '1GB';
ALTER SYSTEM SET
 max_wal_size = '4GB';

sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
sudo yum -y repolist
sudo yum install postgresql15-contrib

sudo /usr/pgsql-15/bin/postgresql-15-setup initdb
systemctl enable postgresql-15.service
systemctl status postgresql-15.service
systemctl restart postgresql-15.service

create user gpadmin;
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

for f in *.csv*; do psql -U postgres -d test -c "\\COPY taxi_trips FROM PROGRAM 'cat $f' CSV HEADER"; done
</pre>






<pre>
---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
-----------------------------------------------Описать что и как делали и с какими проблемами столкнулись------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
</pre>

траблошутил служба cockroach не стартовала столкнулся неверных прав доступа на сертификатах
следом вылезла проблема с параметром  AllowZoneDrifting в  конфиге /etc/firewalld/firewalld.conf 
cockroach не запустилась пока не поменял параметр с yes на no.

