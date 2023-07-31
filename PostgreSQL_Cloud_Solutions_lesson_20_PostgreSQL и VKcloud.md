<pre>
---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
----------------------------------PostgreSQL и VKcloud, GCP, AWS, ЯО, Sbercloud------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
</pre>


#### регистрация в VKcloud 
#### подготовительные мероприятия создаю vm в VKcloud 
#### Будем создавать мастер + standby в асинхронной репликации



#### 1)

#### Создаю общуюю  виртуальную сеть  для обединения моих vm в кластер ( вкладка  Виртуальные сети --> Сети --> создать сеть 10.0.0.0/24)



#### 2)

#### нарезаю две VM  характеристики 1cpu 2озу 10 hdd ( обязательно для начала привязываю vm к внутренней сети а после подключаю внешний ip)
#### обрашаю внимание на Настройки Firewall (очень экстровагантно)

#### команда на подключение к VM (с предварительно сгенирированным ключем)
<pre>
ssh -i C:\Users\Adm\.ssh\vk_cloud_otus.pem centos@212.233.88.39    внутр ip 10.0.0.22
ssh -i C:\Users\Adm\.ssh\vk_cloud_otus.pem centos@212.233.88.39
ssh -i C:\Users\Adm\.ssh\vk_cloud_otus.pem centos@212.233.88.39


ssh -i C:\Users\Adm\.ssh\vk_cloud_otus.pem centos@37.139.35.232   внутр ip 10.0.0.14
ssh -i C:\Users\Adm\.ssh\vk_cloud_otus.pem centos@37.139.35.232
ssh -i C:\Users\Adm\.ssh\vk_cloud_otus.pem centos@37.139.35.232

Ппимечание:
Выберите имя пользователя
Определите имя пользователя (логин) операционной системы, которая развернута на целевой ВМ.
В образах VK Cloud (кроме Bitrix) учетная запись root заблокирована в целях безопасности и добавлена учетная запись для использования по умолчанию:

документация
https://mcs.mail.ru/docs/base/iaas/instructions/vm/vm-connect/vm-connect-nix
</pre>


#### 1) 

#### устанавливаю инструменты на каждой их VM 

<pre>
sudo yum -y install epel-release && yum install -y htop mc vim wget telnet git
sudo yum -y install epel-release && yum install -y htop mc vim wget telnet git
sudo yum -y install epel-release && yum install -y htop mc vim wget telnet git

sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
</pre>


#### 2)

#### инициализируем Postgres на каждой vm

<pre>
sudo yum -y install postgresql15-contrib 
sudo /usr/pgsql-15/bin/postgresql-15-setup initdb 
sudo systemctl enable --now postgresql-15
systemctl status postgresql-15
systemctl restart postgresql-15
</pre>


#### 3)

#### настраиваю потоковую репликацию 

<pre>
#### применяю 
sudo -u postgres psql -c "ALTER SYSTEM SET wal_level = 'replica';"
sudo -u postgres psql -c "ALTER SYSTEM SET synchronous_commit TO 'on';"
sudo -u postgres psql -c "ALTER SYSTEM SET listen_addresses to '*';"
sudo systemctl restart postgresql-15

#### создаю на мастере пользователя который будет отвечать за репликацию между VM1 и VM2
sudo -u postgres psql -c "CREATE ROLE user_replicator WITH REPLICATION PASSWORD 'klJlkdfsjImdsksdmsd98' LOGIN;"
добавляю првава на доступ в pg_hba.conf
host    replication     all             10.0.0.0/24               scram-sha-256

#### создаю слот на мастере 
sudo -u postgres psql -c "SELECT * FROM pg_create_physical_replication_slot('my_slot_replication');"

#### проверяю слот
sudo -u postgres psql -c "SELECT * FROM pg_replication_slots;" \gx

SELECT * FROM pg_replication_slots \gx
postgres=# SELECT * FROM pg_replication_slots \gx
-[ RECORD 1 ]-------+--------------------
slot_name           | my_slot_replication
plugin              |
slot_type           | physical
datoid              |
database            |
temporary           | f
active              | f
active_pid          |
xmin                |
catalog_xmin        |
restart_lsn         |
confirmed_flush_lsn |
wal_status          |
safe_wal_size       |
two_phase           | f



#на VM2 останавливаю кластер удаляю данные 
sudo systemctl stop postgresql-15
sudo -u postgres rm -rf /var/lib/pgsql/15/data/*
sudo -u postgres pg_basebackup -h 10.0.0.22 -D /var/lib/pgsql/15/data -U user_replicator -P -v -R -X stream -S my_slot_replication
could not change directory to "/home/centos": Permission denied
Password:
pg_basebackup: initiating base backup, waiting for checkpoint to complete
pg_basebackup: checkpoint completed
pg_basebackup: write-ahead log start point: 0/2000060 on timeline 1
pg_basebackup: starting background WAL receiver
23492/23492 kB (100%), 1/1 tablespace
pg_basebackup: write-ahead log end point: 0/2000138
pg_basebackup: waiting for background process to finish streaming ...
pg_basebackup: syncing data to disk ...
pg_basebackup: renaming backup_manifest.tmp to backup_manifest
pg_basebackup: base backup completed

проверяю что появилась дополнителная информация по слоту репликации
cat /var/lib/pgsql/15/data/postgresql.auto.conf

#### запускаю кластер на VM2 Сейчас VM4сервер является репликой (находится в режиме восстановления):
sudo systemctl restart postgresql-15
sudo -u postgres psql -c "SELECT pg_is_in_recovery();"


##### на главном сервере VM1 проверяем 
Проверка физической репликации  sync_state: sync говорит о том, что репликация работает в асинхронном режиме.
postgres=# SELECT * FROM pg_stat_replication \gx
-[ RECORD 1 ]----+------------------------------
pid              | 8778
usesysid         | 16388
usename          | user_replicator
application_name | walreceiver
client_addr      | 10.0.0.14
client_hostname  |
client_port      | 57796
backend_start    | 2023-07-31 13:26:18.655922+00
backend_xmin     |
state            | streaming
sent_lsn         | 0/3000060
write_lsn        | 0/3000060
flush_lsn        | 0/3000060
replay_lsn       | 0/3000060
write_lag        |
flush_lag        |
replay_lag       |
sync_priority    | 0
sync_state       | async
reply_time       | 2023-07-31 13:27:28.758617+00



##### Репликация работает все ок

#### для начала я прогнал  кластер postgres на настройках по умолчанию уттилитой 
#### pgbench и sysbench чтобы было с чем сравнивать производительностьсть в других облаках
#### Используем pgbench на начальных настройках кластера postgres

<pre>
описание ключей:

c clients
--client=clients Количество смоделированных клиентов, то есть количество одновременных сеансов базы данных. По умолчанию 1.

-C (каждый пользователь подключается под своей сессией)

-P sec
--progress=sec
Показывать отчет о проделанной работе каждую sec секунду. Отчет включает время с начала выполнения, TPS с момента последнего отчета, а также среднюю задержку транзакции и стандартное отклонение с момента последнего отчета. При регулировании ( -R) задержка вычисляется относительно запланированного времени начала транзакции, а не фактического времени начала транзакции, поэтому она также включает среднее время задержки расписания.

-T seconds
--time=seconds
Запустите тест на это количество секунд, а не на фиксированное количество транзакций на клиента. -t и -Tявляются взаимоисключающими.

Инициализация тестовой базы
/usr/pgsql-15/bin/pgbench -i postgres -U postgres
dropping old tables...
NOTICE:  table "pgbench_accounts" does not exist, skipping
NOTICE:  table "pgbench_branches" does not exist, skipping
NOTICE:  table "pgbench_history" does not exist, skipping
NOTICE:  table "pgbench_tellers" does not exist, skipping
creating tables...
generating data (client-side)...
100000 of 100000 tuples (100%) done (elapsed 0.10 s, remaining 0.00 s)
vacuuming...
creating primary keys...
done in 0.46 s (drop tables 0.00 s, create tables 0.04 s, client-side generate 0.24 s, vacuum 0.05 s, primary keys 0.13 s).

И собственно запуск самого теста получаем выхлоп на настройках по умолчанию
/usr/pgsql-15/bin/pgbench -c8 -P 60 -T 600 -U postgres postgres
pgbench (15.3)
starting vacuum...end.
progress: 60.0 s, 148.8 tps, lat 53.717 ms stddev 14.903, 0 failed
progress: 120.0 s, 143.6 tps, lat 55.703 ms stddev 27.166, 0 failed
progress: 180.0 s, 148.1 tps, lat 54.016 ms stddev 18.425, 0 failed
progress: 240.0 s, 145.8 tps, lat 54.826 ms stddev 23.546, 0 failed
progress: 300.0 s, 146.7 tps, lat 54.553 ms stddev 24.426, 0 failed
progress: 360.0 s, 148.1 tps, lat 54.016 ms stddev 17.758, 0 failed
progress: 420.0 s, 147.6 tps, lat 54.185 ms stddev 21.161, 0 failed
progress: 480.0 s, 147.0 tps, lat 54.422 ms stddev 21.949, 0 failed
progress: 540.0 s, 145.8 tps, lat 54.864 ms stddev 24.537, 0 failed
progress: 600.0 s, 146.5 tps, lat 54.598 ms stddev 23.723, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
maximum number of tries: 1
duration: 600 s
number of transactions actually processed: 88083
number of failed transactions: 0 (0.000%)
latency average = 54.485 ms
latency stddev = 22.021 ms
initial connection time = 28.628 ms
tps = 146.799855 (without initial connection time)
</pre>


#### sysbench на начальных настройках кластера postgres
curl -s https://packagecloud.io/install/repositories/akopytov/sysbench/script.rpm.sh | sudo bash
sudo yum -y install sysbench

#### подготовка
sudo -u postgres psql -c "CREATE ROLE test LOGIN SUPERUSER PASSWORD 'test'"
sudo -u postgres psql -c "CREATE DATABASE test"
sudo -u postgres sysbench \
--db-driver=pgsql \
--oltp-table-size=1000000 \
--oltp-tables-count=23 \
--threads=1 \
--pgsql-host=localhost \
--pgsql-port=5432 \
--pgsql-user=test \
--pgsql-password=test \
--pgsql-db=test \
/usr/share/sysbench/tests/include/oltp_legacy/parallel_prepare.lua \
run

#### получили базу test 23 таблицы обьем 5520 MB 

#### получаем выхлоп на настройках по умолчанию
sysbench \
--db-driver=pgsql \
--report-interval=60 \
--oltp-table-size=1000000 \
--oltp-tables-count=23 \
--threads=64 \
--time=600 \
--pgsql-host=localhost \
--pgsql-port=5432 \
--pgsql-user=test \
--pgsql-password=test \
--pgsql-db=test \
/usr/share/sysbench/tests/include/oltp_legacy/oltp.lua \
run

#### результат


</pre>