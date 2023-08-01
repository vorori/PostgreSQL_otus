

##### PostgreSQL и VKcloud, GCP, AWS, ЯО, Sbercloud-

<pre>
---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------VKcloud-----------------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
</pre>


#### регистрация в VKcloud https://mcs.mail.ru/app/auth
#### подготовительные мероприятия создаю vm в VKcloud 
#### Будем создавать мастер + standby в асинхронной репликации



#### 1)

#### Создаю общуюю  виртуальную сеть  для обединения моих vm в кластер ( вкладка  Виртуальные сети --> Сети --> создать сеть 10.0.0.0/24)



#### 1.2)

#### нарезаю две VM  характеристики 2cpu 2озу 10 hdd ( обязательно для начала привязываю vm к внутренней сети а после подключаю внешний ip)
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


#### 1.3) 

#### устанавливаю инструменты на каждой их VM 

<pre>
sudo yum -y install epel-release && yum install -y htop mc vim wget telnet git
sudo yum -y install epel-release && yum install -y htop mc vim wget telnet git
sudo yum -y install epel-release && yum install -y htop mc vim wget telnet git

sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
</pre>


#### 1.4)

#### инициализируем Postgres на каждой vm

<pre>
sudo yum -y install postgresql15-contrib 
sudo /usr/pgsql-15/bin/postgresql-15-setup initdb 
sudo systemctl enable --now postgresql-15
systemctl status postgresql-15
systemctl restart postgresql-15
</pre>


#### 1.5)

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
</pre>


#### на VM2 останавливаю кластер удаляю данные 

<pre>
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

#### запускаю кластер на VM2 Сейчас сервер является репликой (находится в режиме восстановления):
sudo systemctl restart postgresql-15
sudo -u postgres psql -c "SELECT pg_is_in_recovery();"
</pre>


##### на главном сервере VM1 проверяем 

<pre>
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
</pre>


#### 1.6

#### для начала я прогнал  кластер postgres на настройках по умолчанию уттилитой 
#### pgbench и sysbench чтобы было с чем сравнивать производительностьсть в других облаках
#### Используем pgbench на начальных настройках кластера postgres
#### результат для VKcloud

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


#### 1.7

#### sysbench на начальных настройках кластера postgres

<pre>
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
#### результат для VKcloud
[ 60s ] thds: 64 tps: 23.17 qps: 476.76 (r/w/o: 335.72/93.62/47.41) lat (ms,95%): 6594.16 err/s: 0.00 reconn/s: 0.00
[ 120s ] thds: 64 tps: 29.67 qps: 589.79 (r/w/o: 412.59/117.87/59.33) lat (ms,95%): 4128.91 err/s: 0.00 reconn/s: 0.00
[ 180s ] thds: 64 tps: 34.50 qps: 697.42 (r/w/o: 488.03/140.38/69.00) lat (ms,95%): 3267.19 err/s: 0.00 reconn/s: 0.00
[ 240s ] thds: 64 tps: 38.07 qps: 761.88 (r/w/o: 533.38/152.32/76.18) lat (ms,95%): 3208.88 err/s: 0.02 reconn/s: 0.00
[ 300s ] thds: 64 tps: 36.08 qps: 718.54 (r/w/o: 502.74/143.63/72.17) lat (ms,95%): 3267.19 err/s: 0.00 reconn/s: 0.00
[ 360s ] thds: 64 tps: 33.68 qps: 674.26 (r/w/o: 472.46/134.43/67.37) lat (ms,95%): 3639.94 err/s: 0.00 reconn/s: 0.00
[ 420s ] thds: 64 tps: 34.00 qps: 682.92 (r/w/o: 477.85/137.03/68.03) lat (ms,95%): 3639.94 err/s: 0.02 reconn/s: 0.00
[ 480s ] thds: 64 tps: 33.08 qps: 658.57 (r/w/o: 461.05/131.35/66.17) lat (ms,95%): 3574.99 err/s: 0.00 reconn/s: 0.00
[ 540s ] thds: 64 tps: 30.67 qps: 612.22 (r/w/o: 428.42/122.47/61.33) lat (ms,95%): 3982.86 err/s: 0.00 reconn/s: 0.00
[ 600s ] thds: 64 tps: 32.87 qps: 661.20 (r/w/o: 462.70/132.77/65.73) lat (ms,95%): 3706.08 err/s: 0.00 reconn/s: 0.00
SQL statistics:
    queries performed:
        read:                            274596
        write:                           78451
        other:                           39229
        total:                           392276
    transactions:                        19612  (32.61 per sec.)
    queries:                             392276 (652.30 per sec.)
    ignored errors:                      2      (0.00 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          601.3743s
    total number of events:              19612

Latency (ms):
         min:                                    3.53
         avg:                                 1960.66
         max:                                12411.53
         95th percentile:                     3706.08
         sum:                             38452446.26

Threads fairness:
    events (avg/stddev):           306.4375/6.81
    execution time (avg/stddev):   600.8195/0.44

</pre>



<pre>
---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------Sbercloud---------------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
</pre>

#### регистрация в Sbercloud https://cloud.ru/ru
#### Будем создавать мастер + standby в асинхронной репликации настройку я опускаю т.к. она идентична вышеописсанной на примере с VKcloud
#### подготовительные мероприятия создаю vm в Sbercloud

<pre>
Инструкция подключения
1. Активация консоли https://console.hc.sbercloud.ru/console/?region=ru-moscow-1#/home( пополнение счета)
2. Создание сети https://cloud.ru/ru/docs/vpc/ug/topics/managing-networks__create-vpc-network.html Перейдите в раздел Network → Virtual Private Cloud.
3. документация по созданию VM https://cloud.ru/ru/docs/ecs/ug/index.html
4. создание Elastic Cloud Server VM https://console.hc.sbercloud.ru/ecm/?agencyId=c7QXyl2IUo9XfqpunlzXoreUPd9KK3hq&region=ru-moscow-1&locale=en-us#/ecs/manager/vmList
5. подключение по ssh root@37.230.196.5    ssh root@37.230.196.77
</pre>

#### Используем pgbench 
#### результат для Sbercloud
<pre>
/usr/pgsql-15/bin/pgbench -c8 -P 60 -T 600 -U postgres postgres
pgbench (15.3)
starting vacuum...end.
progress: 60.0 s, 892.4 tps, lat 5.741 ms stddev 1.678, 0 failed
progress: 120.0 s, 706.7 tps, lat 5.684 ms stddev 2.255, 0 failed
progress: 180.0 s, 664.1 tps, lat 5.862 ms stddev 1.824, 0 failed
progress: 240.0 s, 891.2 tps, lat 5.747 ms stddev 1.612, 0 failed
progress: 300.0 s, 839.7 tps, lat 5.553 ms stddev 1.561, 0 failed
progress: 360.0 s, 720.2 tps, lat 5.630 ms stddev 1.693, 0 failed
progress: 420.0 s, 752.1 tps, lat 5.506 ms stddev 1.638, 0 failed
progress: 420.0 s, 752.1 tps, lat 5.506 ms stddev 1.638, 0 failed
progress: 480.0 s, 831.7 tps, lat 5.585 ms stddev 1.624, 0 failed
progress: 540.0 s, 639.4 tps, lat 5.555 ms stddev 1.606, 0 failed
progress: 600.0 s, 875.5 tps, lat 5.419 ms stddev 1.456, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
maximum number of tries: 1
duration: 600 s
number of transactions actually processed: 852780
number of failed transactions: 0 (0.000%)
latency average = 5.625 ms
latency stddev = 1.709 ms
initial connection time = 13.055 ms
tps = 721.303039 (without initial connection time)
</pre>



#### Используем sysbench 
#### результат для Sbercloud

<pre>
[ 60s ] thds: 64 tps: 219.72 qps: 4406.44 (r/w/o: 3086.43/879.40/440.61) lat (ms,95%): 773.68 err/s: 0.05 reconn/s: 0.00
[ 120s ] thds: 64 tps: 246.82 qps: 4938.69 (r/w/o: 3457.09/987.86/493.73) lat (ms,95%): 707.07 err/s: 0.03 reconn/s: 0.00
[ 180s ] thds: 64 tps: 248.84 qps: 4976.15 (r/w/o: 3483.87/994.49/497.79) lat (ms,95%): 669.89 err/s: 0.03 reconn/s: 0.00
[ 240s ] thds: 64 tps: 271.57 qps: 5431.51 (r/w/o: 3801.84/1086.44/543.23) lat (ms,95%): 669.89 err/s: 0.03 reconn/s: 0.00
[ 300s ] thds: 64 tps: 298.60 qps: 5975.13 (r/w/o: 4183.38/1194.48/597.27) lat (ms,95%): 634.66 err/s: 0.02 reconn/s: 0.00
[ 360s ] thds: 64 tps: 312.05 qps: 6241.52 (r/w/o: 4369.04/1248.32/624.15) lat (ms,95%): 590.56 err/s: 0.02 reconn/s: 0.00
[ 420s ] thds: 64 tps: 304.57 qps: 6091.76 (r/w/o: 4263.50/1219.08/609.19) lat (ms,95%): 623.33 err/s: 0.02 reconn/s: 0.00
[ 480s ] thds: 64 tps: 306.47 qps: 6130.43 (r/w/o: 4291.40/1226.01/613.03) lat (ms,95%): 590.56 err/s: 0.02 reconn/s: 0.00
[ 540s ] thds: 64 tps: 303.95 qps: 6078.69 (r/w/o: 4255.30/1215.37/608.03) lat (ms,95%): 580.02 err/s: 0.05 reconn/s: 0.00
[ 600s ] thds: 64 tps: 303.68 qps: 6074.08 (r/w/o: 4252.53/1214.07/607.48) lat (ms,95%): 590.56 err/s: 0.05 reconn/s: 0.00
SQL statistics:
    queries performed:
        read:                            2366826
        write:                           676180
        other:                           338136
        total:                           3381142
    transactions:                        169040 (281.64 per sec.)
    queries:                             3381142 (5633.28 per sec.)
    ignored errors:                      19     (0.03 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          600.2072s
    total number of events:              169040

Latency (ms):
         min:                                    2.91
         avg:                                  227.21
         max:                                 1635.16
         95th percentile:                      634.66
         sum:                             38407075.33

Threads fairness:
    events (avg/stddev):           2641.2500/9.07
    execution time (avg/stddev):   600.1106/0.05
</pre>



<pre>
---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------Yandex Cloud------------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
</pre>


#### Будем создавать мастер + standby в асинхронной репликации настройку я опускаю т.к. она идентична вышеописсанной на примере с VKcloud
#### Подготовительные мероприятия нарезаю vm-ки идентичные по характеристикам выше (2cpu 2озу 10 hdd)

#### Используем pgbench 
#### результат для Yandex Cloud

<pre>
progress: 60.0 s, 652.1 tps, lat 12.260 ms stddev 10.435
progress: 120.0 s, 479.9 tps, lat 16.663 ms stddev 14.426
progress: 180.0 s, 348.1 tps, lat 22.984 ms stddev 21.320
progress: 240.0 s, 600.8 tps, lat 13.316 ms stddev 12.299
progress: 300.0 s, 612.3 tps, lat 13.064 ms stddev 12.570
progress: 360.0 s, 576.5 tps, lat 13.874 ms stddev 14.898
progress: 420.0 s, 608.2 tps, lat 13.154 ms stddev 11.765
progress: 480.0 s, 628.9 tps, lat 12.721 ms stddev 11.093
progress: 540.0 s, 575.1 tps, lat 13.910 ms stddev 11.777
progress: 600.0 s, 372.1 tps, lat 21.497 ms stddev 23.477
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
duration: 600 s
number of transactions actually processed: 327245
latency average = 14.666 ms
latency stddev = 14.491 ms
initial connection time = 16.419 ms
tps = 545.411638 (without initial connection time)
</pre>


#### Используем sysbench 
#### результат для Yandex Cloud

<pre>
[ 5s ] thds: 64 tps: 92.13 qps: 2049.52 (r/w/o: 1451.55/399.91/198.06) lat (ms,95%): 1280.93 err/s: 0.40 reconn/s: 0.00
[ 10s ] thds: 64 tps: 82.61 qps: 1659.93 (r/w/o: 1164.69/330.03/165.21) lat (ms,95%): 2045.74 err/s: 0.00 reconn/s: 0.00
[ 15s ] thds: 64 tps: 48.00 qps: 934.53 (r/w/o: 652.75/185.79/95.99) lat (ms,95%): 2493.86 err/s: 0.00 reconn/s: 0.00
[ 20s ] thds: 64 tps: 89.20 qps: 1789.90 (r/w/o: 1252.47/359.02/178.41) lat (ms,95%): 1561.52 err/s: 0.00 reconn/s: 0.00
[ 25s ] thds: 64 tps: 31.40 qps: 646.39 (r/w/o: 453.19/130.40/62.80) lat (ms,95%): 3448.53 err/s: 0.00 reconn/s: 0.00
[ 30s ] thds: 64 tps: 32.60 qps: 602.00 (r/w/o: 425.60/111.20/65.20) lat (ms,95%): 3911.79 err/s: 0.00 reconn/s: 0.00
[ 35s ] thds: 64 tps: 113.20 qps: 2324.46 (r/w/o: 1617.84/480.21/226.41) lat (ms,95%): 1191.92 err/s: 0.00 reconn/s: 0.00
[ 40s ] thds: 64 tps: 41.80 qps: 815.99 (r/w/o: 576.19/156.20/83.60) lat (ms,95%): 2449.36 err/s: 0.00 reconn/s: 0.00
[ 45s ] thds: 64 tps: 33.00 qps: 653.00 (r/w/o: 457.80/129.20/66.00) lat (ms,95%): 3386.99 err/s: 0.00 reconn/s: 0.00
[ 50s ] thds: 64 tps: 32.80 qps: 669.01 (r/w/o: 464.21/139.00/65.80) lat (ms,95%): 4055.23 err/s: 0.00 reconn/s: 0.00
[ 55s ] thds: 64 tps: 32.40 qps: 666.80 (r/w/o: 465.20/136.80/64.80) lat (ms,95%): 3706.08 err/s: 0.00 reconn/s: 0.00
[ 60s ] thds: 64 tps: 37.80 qps: 734.40 (r/w/o: 514.40/144.40/75.60) lat (ms,95%): 3326.55 err/s: 0.00 reconn/s: 0.00
SQL statistics:
    queries performed:
        read:                            47614
        write:                           13598
        other:                           6804
        total:                           68016
    transactions:                        3399   (53.50 per sec.)
    queries:                             68016  (1070.47 per sec.)
    ignored errors:                      2      (0.03 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          63.5364s
    total number of events:              3399

Latency (ms):
         min:                                   10.04
         avg:                                 1147.50
         max:                                 6655.73
         95th percentile:                     2828.87
         sum:                              3900353.37

Threads fairness:
    events (avg/stddev):           53.1094/3.73
    execution time (avg/stddev):   60.9430/0.63
</pre>


<pre>
---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
--------------------------------------------------------ИТОГО--------------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
</pre>

<pre>
Sbercloud показал себя наииболее производительнее по идентичным тестам из коробки pgbench, sysbench 
Почетное второе место занимает Yandex Cloud
И самое низкие оценки у меня получились у VKcloud

Очень специфичная  документация VKcloud и Sbercloud требуется предварительно прочитать а потом только начинать создавать нужную для нас инфру натыкать в первый раз без доки не получилось
были проблемы с доступнустью vm настройки firewall и настройками обшей сетевой инфраструктуры для репликации. Еше при создании идентичных vm на VKcloud была проблема с установкой тех же самых пакетов
которые на другой вм влетели/установились без проблем сидел вдуплял в это все дело немог понять что это вдург случилось в обшем совой "полтергейст")
Yandex Cloud в этом смысле выигрывает за счет более привычного интерфейса чтоли инструментария но как говорится на любителя ...
</pre>