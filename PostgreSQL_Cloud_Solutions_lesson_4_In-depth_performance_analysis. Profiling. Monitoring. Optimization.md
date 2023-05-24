#### 1)

#### Развернуть Постгрес на ВМ

<pre>
создал vm centos 4 core 4gb озу

sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
sudo yum -y repolist
sudo yum install postgresql15-contrib

sudo /usr/pgsql-15/bin/postgresql-15-setup initdb
systemctl enable postgresql-15.service
systemctl status postgresql-15.service
systemctl restart postgresql-15.service
</pre>

#### 2)

#### Протестировать pg_bench на настройках п умолчанию кластера postgres

<pre>
первый тест запускаю на настройках по умолчанию с postgresql-15

Инициализация тестовой базы и запуск теста
su - postgres
/usr/pgsql-15/bin/pgbench -i postgres -U postgres
/usr/pgsql-15/bin/pgbench -c30 -C -P 60 -T 600 -U postgres postgres

pgbench (15.3)
starting vacuum...end.
progress: 60.0 s, 93.8 tps, lat 310.406 ms stddev 325.807, 0 failed
progress: 120.0 s, 57.7 tps, lat 511.141 ms stddev 545.342, 0 failed
progress: 180.0 s, 92.4 tps, lat 318.340 ms stddev 339.707, 0 failed
progress: 240.0 s, 93.3 tps, lat 315.040 ms stddev 308.005, 0 failed
progress: 300.0 s, 95.7 tps, lat 307.013 ms stddev 338.363, 0 failed
progress: 360.0 s, 96.3 tps, lat 303.817 ms stddev 333.616, 0 failed
progress: 420.0 s, 104.8 tps, lat 281.678 ms stddev 302.016, 0 failed
progress: 480.0 s, 110.2 tps, lat 266.495 ms stddev 265.656, 0 failed
progress: 540.0 s, 111.4 tps, lat 263.816 ms stddev 262.515, 0 failed
progress: 600.0 s, 111.5 tps, lat 263.475 ms stddev 284.337, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 30
number of threads: 1
maximum number of tries: 1
duration: 600 s
number of transactions actually processed: 58059
number of failed transactions: 0 (0.000%)
latency average = 303.688 ms
latency stddev = 330.130 ms
average connection time = 6.301 ms
tps = 96.745819 (including reconnection times)



описание ключей pgbench:

c clients
--client=clients Количество смоделированных клиентов, то есть количество одновременных сеансов базы данных. По умолчанию 1.

-C (каждый пользователь подключается под своей сессией)

-P sec
--progress=sec
Показывать отчет о проделанной работе каждую sec секунду. Отчет включает время с начала выполнения, 
TPS с момента последнего отчета, а также среднюю задержку транзакции и стандартное отклонение с момента последнего отчета. 
При регулировании ( -R) задержка вычисляется относительно запланированного времени начала транзакции, 
а не фактического времени начала транзакции, поэтому она также включает среднее время задержки расписания.

-T seconds
--time=seconds
Запустите тест на это количество секунд, а не на фиксированное количество транзакций на клиента. -t и -T являются взаимоисключающими.
</pre>





#### Протестировать sysbench на настройках п умолчанию кластера postgres

<pre>
curl -s https://packagecloud.io/install/repositories/akopytov/sysbench/script.rpm.sh | sudo bash
sudo yum -y install sysbench


#подготовка
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


# получили базу test 23 таблицы обьем 5586  MB 
test      | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            |                       | 5586 MB | pg_default |


postgres=# \c test
You are now connected to database "test" as user "postgres".
test=# \dt
         List of relations
 Schema |   Name   | Type  | Owner
--------+----------+-------+-------
 public | sbtest1  | table | test
 public | sbtest10 | table | test
 public | sbtest11 | table | test
 public | sbtest12 | table | test
 public | sbtest13 | table | test
 public | sbtest14 | table | test
 public | sbtest15 | table | test
 public | sbtest16 | table | test
 public | sbtest17 | table | test
 public | sbtest18 | table | test
 public | sbtest19 | table | test
 public | sbtest2  | table | test
 public | sbtest20 | table | test
 public | sbtest21 | table | test
 public | sbtest22 | table | test
 public | sbtest23 | table | test
 public | sbtest3  | table | test
 public | sbtest4  | table | test
 public | sbtest5  | table | test
 public | sbtest6  | table | test
 public | sbtest7  | table | test
 public | sbtest8  | table | test
 public | sbtest9  | table | test
(23 rows)



#запускаем сам тест
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

# количество подключенных ссессий к кластеру postgre
postgres=# select count(*) as connections, usename from pg_stat_activity group by usename order by count(*) desc;
 connections | usename
-------------+----------
          64 | test
           4 |
           2 | postgres



# результат
Running the test with following options:
Number of threads: 64
Report intermediate results every 60 second(s)
Initializing random number generator from current time


Initializing worker threads...

Threads started!

[ 60s ] thds: 64 tps: 277.64 qps: 5567.57 (r/w/o: 3899.34/1111.85/556.37) lat (ms,95%): 511.33 err/s: 0.00 reconn/s: 0.00
[ 120s ] thds: 64 tps: 373.36 qps: 7466.75 (r/w/o: 5226.91/1493.07/746.78) lat (ms,95%): 320.17 err/s: 0.00 reconn/s: 0.00
[ 180s ] thds: 64 tps: 394.43 qps: 7887.91 (r/w/o: 5521.44/1577.53/788.95) lat (ms,95%): 297.92 err/s: 0.02 reconn/s: 0.00
[ 240s ] thds: 64 tps: 380.01 qps: 7602.66 (r/w/o: 5321.44/1521.04/760.18) lat (ms,95%): 308.84 err/s: 0.05 reconn/s: 0.00
[ 300s ] thds: 64 tps: 371.71 qps: 7428.69 (r/w/o: 5199.66/1485.59/743.44) lat (ms,95%): 320.17 err/s: 0.02 reconn/s: 0.00
[ 360s ] thds: 64 tps: 368.12 qps: 7363.21 (r/w/o: 5153.99/1472.90/736.32) lat (ms,95%): 325.98 err/s: 0.00 reconn/s: 0.00
[ 420s ] thds: 64 tps: 379.66 qps: 7597.42 (r/w/o: 5319.79/1518.20/759.43) lat (ms,95%): 308.84 err/s: 0.03 reconn/s: 0.00
[ 480s ] thds: 64 tps: 354.76 qps: 7095.02 (r/w/o: 4966.06/1419.34/709.63) lat (ms,95%): 337.94 err/s: 0.03 reconn/s: 0.00
[ 540s ] thds: 64 tps: 352.75 qps: 7056.53 (r/w/o: 4939.95/1411.07/705.51) lat (ms,95%): 344.08 err/s: 0.00 reconn/s: 0.00
[ 600s ] thds: 64 tps: 332.88 qps: 6655.80 (r/w/o: 4658.61/1331.42/665.78) lat (ms,95%): 363.18 err/s: 0.02 reconn/s: 0.00
SQL statistics:
    queries performed:
        read:                            3012716
        write:                           860735
        other:                           430409
        total:                           4303860
    transactions:                        215184 (358.46 per sec.)
    queries:                             4303860 (7169.44 per sec.)
    ignored errors:                      10     (0.02 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          600.3043s
    total number of events:              215184

Latency (ms):
         min:                                   10.22
         avg:                                  178.49
         max:                                 1852.58
         95th percentile:                      337.94
         sum:                             38407818.58

Threads fairness:
    events (avg/stddev):           3362.2500/23.83
    execution time (avg/stddev):   600.1222/0.09

</pre>





#### 3)

#### Выставить оптимальные настройки

<pre>
применил настройки:

ALTER SYSTEM SET
 max_connections = '100';
ALTER SYSTEM SET
 shared_buffers = '1GB';
ALTER SYSTEM SET
 effective_cache_size = '3GB';
ALTER SYSTEM SET
 maintenance_work_mem = '256MB';
ALTER SYSTEM SET
 checkpoint_completion_target = '0.9';
ALTER SYSTEM SET
 wal_buffers = '16MB';
ALTER SYSTEM SET
 default_statistics_target = '100';
ALTER SYSTEM SET
 random_page_cost = '1.1';
ALTER SYSTEM SET
 effective_io_concurrency = '200';
ALTER SYSTEM SET
 work_mem = '5242kB';
ALTER SYSTEM SET
 min_wal_size = '2GB';
ALTER SYSTEM SET
 max_wal_size = '8GB';
ALTER SYSTEM SET
 max_worker_processes = '4';
ALTER SYSTEM SET
 max_parallel_workers_per_gather = '2';
ALTER SYSTEM SET
 max_parallel_workers = '4';
ALTER SYSTEM SET
 max_parallel_maintenance_workers = '2';
</pre>

#### 4)

####  Проверить насколько выросла производительность


#### pg_bench тест на опьтимальных настройках

<pre>
/usr/pgsql-15/bin/pgbench -c30 -C -P 60 -T 600 -U postgres postgres

# количество подключенных ссессий к кластеру postgre
postgres=# select count(*) as connections, usename from pg_stat_activity group by usename order by count(*) desc;
 connections | usename
-------------+----------
          31 | postgres
           5 |
(2 rows)

pgbench (15.3)
starting vacuum...end.
progress: 60.0 s, 112.1 tps, lat 260.489 ms stddev 280.231, 0 failed
progress: 120.0 s, 116.0 tps, lat 253.652 ms stddev 257.345, 0 failed
progress: 180.0 s, 117.6 tps, lat 250.132 ms stddev 246.070, 0 failed
progress: 240.0 s, 119.8 tps, lat 245.140 ms stddev 247.512, 0 failed
progress: 300.0 s, 121.1 tps, lat 242.780 ms stddev 255.479, 0 failed
progress: 360.0 s, 121.5 tps, lat 241.907 ms stddev 248.678, 0 failed
progress: 420.0 s, 120.7 tps, lat 243.169 ms stddev 243.779, 0 failed
progress: 480.0 s, 121.5 tps, lat 242.319 ms stddev 231.965, 0 failed
progress: 540.0 s, 117.2 tps, lat 250.436 ms stddev 261.228, 0 failed
progress: 600.0 s, 116.9 tps, lat 250.989 ms stddev 259.774, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 30
number of threads: 1
maximum number of tries: 1
duration: 600 s
number of transactions actually processed: 71095
number of failed transactions: 0 (0.000%)
latency average = 248.043 ms
latency stddev = 253.474 ms
average connection time = 5.110 ms
tps = 118.471823 (including reconnection times)
</pre>




#### sysbench тест на опьтимальных настройках

<pre>
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

# количество подключенных ссессий к кластеру postgre
postgres=# select count(*) as connections, usename from pg_stat_activity group by usename order by count(*) desc;
 connections | usename
-------------+----------
          64 | test
           4 |
           2 | postgres
(3 rows)

Initializing worker threads...

Threads started!

[ 60s ] thds: 64 tps: 340.36 qps: 6825.19 (r/w/o: 4779.01/1364.31/681.87) lat (ms,95%): 350.33 err/s: 0.02 reconn/s: 0.00
[ 120s ] thds: 64 tps: 367.92 qps: 7355.23 (r/w/o: 5149.58/1469.81/735.84) lat (ms,95%): 325.98 err/s: 0.00 reconn/s: 0.00
[ 180s ] thds: 64 tps: 355.69 qps: 7111.60 (r/w/o: 4976.91/1423.24/711.45) lat (ms,95%): 344.08 err/s: 0.05 reconn/s: 0.00
[ 240s ] thds: 64 tps: 360.70 qps: 7218.44 (r/w/o: 5054.01/1442.85/721.58) lat (ms,95%): 337.94 err/s: 0.03 reconn/s: 0.00
[ 300s ] thds: 64 tps: 386.12 qps: 7719.99 (r/w/o: 5404.10/1543.64/772.25) lat (ms,95%): 308.84 err/s: 0.00 reconn/s: 0.00
[ 360s ] thds: 64 tps: 388.39 qps: 7769.04 (r/w/o: 5438.26/1553.85/776.93) lat (ms,95%): 308.84 err/s: 0.05 reconn/s: 0.00
[ 420s ] thds: 64 tps: 376.33 qps: 7527.94 (r/w/o: 5269.37/1505.85/752.72) lat (ms,95%): 320.17 err/s: 0.02 reconn/s: 0.00
[ 480s ] thds: 64 tps: 398.57 qps: 7972.45 (r/w/o: 5581.01/1594.24/797.21) lat (ms,95%): 292.60 err/s: 0.00 reconn/s: 0.00
[ 540s ] thds: 64 tps: 376.97 qps: 7538.35 (r/w/o: 5276.97/1507.30/754.08) lat (ms,95%): 320.17 err/s: 0.03 reconn/s: 0.00
[ 600s ] thds: 64 tps: 397.42 qps: 7948.40 (r/w/o: 5563.90/1589.64/794.86) lat (ms,95%): 303.33 err/s: 0.00 reconn/s: 0.00
SQL statistics:
    queries performed:
        read:                            3149818
        write:                           899902
        other:                           449996
        total:                           4499716
    transactions:                        224975 (374.61 per sec.)
    queries:                             4499716 (7492.46 per sec.)
    ignored errors:                      12     (0.02 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          600.5635s
    total number of events:              224975

Latency (ms):
         min:                                    8.87
         avg:                                  170.73
         max:                                 2710.20
         95th percentile:                      320.17
         sum:                             38410101.91

Threads fairness:
    events (avg/stddev):           3515.2344/24.94
    execution time (avg/stddev):   600.1578/0.12



#### Итого : pg_bench и sysbench на новых оптимизированных параметрах дали небольшой прирост производительности 10% и уменьшилась задержка

</pre>


#### 5)

####  Настроить кластер на оптимальную производительность не обращая внимания на стабильность БД


<pre>

synchronous_commit = off
fsync = off
full_page_writes = off
full_page_writes = off

-bash-4.2$ /usr/pgsql-15/bin/pgbench -c30 -C -P 60 -T 600 -U postgres postgres
pgbench (15.3)
starting vacuum...end.
progress: 60.0 s, 150.9 tps, lat 192.227 ms stddev 185.948, 0 failed
progress: 120.0 s, 151.3 tps, lat 193.043 ms stddev 182.991, 0 failed
progress: 180.0 s, 139.9 tps, lat 207.798 ms stddev 201.862, 0 failed
progress: 240.0 s, 138.6 tps, lat 210.866 ms stddev 211.955, 0 failed
progress: 300.0 s, 137.2 tps, lat 212.815 ms stddev 199.652, 0 failed
progress: 360.0 s, 134.9 tps, lat 215.553 ms stddev 200.597, 0 failed
progress: 420.0 s, 138.1 tps, lat 211.431 ms stddev 197.627, 0 failed
progress: 480.0 s, 133.9 tps, lat 218.225 ms stddev 216.510, 0 failed
progress: 540.0 s, 134.3 tps, lat 216.582 ms stddev 202.281, 0 failed
progress: 600.0 s, 110.9 tps, lat 263.530 ms stddev 286.317, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 30
number of threads: 1
maximum number of tries: 1
duration: 600 s
number of transactions actually processed: 82220
number of failed transactions: 0 (0.000%)
latency average = 212.801 ms
latency stddev = 208.933 ms
average connection time = 6.060 ms
tps = 137.016493 (including reconnection times)




Initializing worker threads...

Threads started!

[ 60s ] thds: 64 tps: 350.18 qps: 7015.26 (r/w/o: 4912.55/1401.22/701.49) lat (ms,95%): 369.77 err/s: 0.03 reconn/s: 0.00
[ 120s ] thds: 64 tps: 385.31 qps: 7707.24 (r/w/o: 5395.16/1541.44/770.65) lat (ms,95%): 344.08 err/s: 0.00 reconn/s: 0.00
[ 180s ] thds: 64 tps: 406.11 qps: 8121.84 (r/w/o: 5685.47/1624.15/812.21) lat (ms,95%): 314.45 err/s: 0.00 reconn/s: 0.00
[ 240s ] thds: 64 tps: 390.05 qps: 7800.72 (r/w/o: 5460.48/1560.08/780.16) lat (ms,95%): 344.08 err/s: 0.00 reconn/s: 0.00
[ 300s ] thds: 64 tps: 398.11 qps: 7963.15 (r/w/o: 5574.24/1592.64/796.26) lat (ms,95%): 325.98 err/s: 0.02 reconn/s: 0.00
[ 360s ] thds: 64 tps: 408.20 qps: 8165.06 (r/w/o: 5715.78/1632.82/816.46) lat (ms,95%): 314.45 err/s: 0.02 reconn/s: 0.00
[ 420s ] thds: 64 tps: 383.31 qps: 7665.60 (r/w/o: 5365.37/1533.58/766.65) lat (ms,95%): 331.91 err/s: 0.00 reconn/s: 0.00
[ 480s ] thds: 64 tps: 363.73 qps: 7272.43 (r/w/o: 5090.64/1454.35/727.43) lat (ms,95%): 356.70 err/s: 0.00 reconn/s: 0.00
[ 540s ] thds: 64 tps: 347.49 qps: 6951.01 (r/w/o: 4865.93/1390.07/695.01) lat (ms,95%): 369.77 err/s: 0.02 reconn/s: 0.00
[ 600s ] thds: 64 tps: 395.48 qps: 7910.83 (r/w/o: 5537.74/1582.11/790.98) lat (ms,95%): 282.25 err/s: 0.02 reconn/s: 0.00
SQL statistics:
    queries performed:
        read:                            3216500
        write:                           918979
        other:                           459509
        total:                           4594988
    transactions:                        229744 (382.81 per sec.)
    queries:                             4594988 (7656.30 per sec.)
    ignored errors:                      6      (0.01 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          600.1556s
    total number of events:              229744

Latency (ms):
         min:                                    5.37
         avg:                                  167.16
         max:                                 1656.14
         95th percentile:                      337.94
         sum:                             38402888.72

Threads fairness:
    events (avg/stddev):           3589.7500/24.35
    execution time (avg/stddev):   600.0451/0.05


#### Итого : pg_bench и sysbench на новых параметрах дали cущественный прирост производительности  задержка уменьшилась еше больше


</pre>

