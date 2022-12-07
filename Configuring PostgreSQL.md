
# 1
развернуть виртуальную машину любым удобным способом
<pre>
создал centos7 sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm

4  cpu 
4  озу
20 hdd
</pre>

# 2
поставить на неё PostgreSQL 14 любым способом

<pre>
sudo yum install postgresql14-contrib 

sudo /usr/pgsql-14/bin/postgresql-14-setup initdb 

sudo systemctl enable --now postgresql-14 

systemctl status postgresql-14

systemctl restart postgresql-14
</pre>





# 3

#для начала я прогнал  кластер postgres на настройках по умолчанию уттилитой 
pgbench и sysbench чтобы было с чем сравнивать

<pre>
# Используем pgbench на начальных настройках кластера postgres

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

/usr/pgsql-14/bin/pgbench -i postgres -U postgres


И собственно запуск самого теста

/usr/pgsql-14/bin/pgbench -c8 -P 60 -T 600 -U postgres postgres

получаем выхлоп на настройках по умолчанию
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


<pre>
#sysbench на начальных настройках кластера postgres

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

# получили базу test 23 таблицы обьем 5520 MB 
\c test
test      | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |                       | 5520 MB | pg_default |

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


#получаем выхлоп на настройках по умолчанию
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

# результат

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



# 4
настроить кластер PostgreSQL 14 на максимальную производительность не
обращая внимание на возможные проблемы с надежностью в случае
аварийной перезагрузки виртуальной машины

<pre>
#применил вот такие настройки

ALTER SYSTEM SET
 max_connections = '100';
ALTER SYSTEM SET
 shared_buffers = '925MB';
ALTER SYSTEM SET
 effective_cache_size = '2775MB';
ALTER SYSTEM SET
 maintenance_work_mem = '236800kB';
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
 work_mem = '4832kB';
ALTER SYSTEM SET
 min_wal_size = '1GB';
ALTER SYSTEM SET
 max_wal_size = '4GB';
</pre>

# 5
нагрузить кластер через утилиту
https://github.com/Percona-Lab/sysbench-tpcc (требует установки
https://github.com/akopytov/sysbench) или через утилиту pgbench (https://postgrespro.ru/docs/postgrespro/14/pgbench)
написать какого значения tps удалось достичь, показать какие параметры какие значения устанавливали и почему

<pre>
заметил что  pgtune советует применить нажеуказанные настройки 
по поводу распаралеливания запросов но из коробки по умолчанию эти параметры завыщены в 2 раза и если их оставить то в моем случае tps был сушественно выше нежели с нижеуказанными настройками.

ALTER SYSTEM SET
 max_worker_processes = '4';
ALTER SYSTEM SET
 max_parallel_workers_per_gather = '2';
ALTER SYSTEM SET
 max_parallel_workers = '4';
ALTER SYSTEM SET
 max_parallel_maintenance_workers = '2';


с новыми настройками получили прирост в tps как pgbench так и в sysbench  по сравнению с первыми тестами на настройках по умолчанию

bash-4.2$ /usr/pgsql-14/bin/pgbench -c8 -P 60 -T 600 -U postgres postgres
pgbench (14.6)
starting vacuum...end.
progress: 60.0 s, 669.0 tps, lat 11.952 ms stddev 9.391
progress: 120.0 s, 648.7 tps, lat 12.331 ms stddev 11.088
progress: 180.0 s, 696.1 tps, lat 11.492 ms stddev 9.197
progress: 240.0 s, 600.6 tps, lat 13.318 ms stddev 11.623
progress: 300.0 s, 639.1 tps, lat 12.517 ms stddev 10.515
progress: 360.0 s, 603.7 tps, lat 13.249 ms stddev 11.932
progress: 420.0 s, 394.1 tps, lat 20.290 ms stddev 21.407
progress: 480.0 s, 559.8 tps, lat 14.294 ms stddev 12.654
progress: 540.0 s, 684.6 tps, lat 11.684 ms stddev 9.671
progress: 600.0 s, 633.4 tps, lat 12.630 ms stddev 11.227
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
duration: 600 s
number of transactions actually processed: 367756
latency average = 13.050 ms
latency stddev = 11.952 ms
initial connection time = 17.225 ms
tps = 612.934325 (without initial connection time)




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

[ 5s ] thds: 64 tps: 105.96 qps: 2266.27 (r/w/o: 1600.74/440.82/224.71) lat (ms,95%): 1506.29 err/s: 0.00 reconn/s: 0.00
[ 10s ] thds: 64 tps: 184.61 qps: 3718.32 (r/w/o: 2605.68/743.42/369.21) lat (ms,95%): 802.05 err/s: 0.00 reconn/s: 0.00
[ 15s ] thds: 64 tps: 246.20 qps: 4924.73 (r/w/o: 3443.95/988.59/492.19) lat (ms,95%): 657.93 err/s: 0.00 reconn/s: 0.00
[ 20s ] thds: 64 tps: 393.42 qps: 7887.10 (r/w/o: 5522.21/1577.66/787.23) lat (ms,95%): 376.49 err/s: 0.00 reconn/s: 0.00
[ 25s ] thds: 64 tps: 567.56 qps: 11341.86 (r/w/o: 7943.08/2263.85/1134.93) lat (ms,95%): 282.25 err/s: 0.00 reconn/s: 0.00
[ 30s ] thds: 64 tps: 680.22 qps: 13574.35 (r/w/o: 9504.85/2709.07/1360.44) lat (ms,95%): 235.74 err/s: 0.00 reconn/s: 0.00
[ 35s ] thds: 64 tps: 199.00 qps: 4048.24 (r/w/o: 2828.03/822.01/398.20) lat (ms,95%): 707.07 err/s: 0.00 reconn/s: 0.00
[ 40s ] thds: 64 tps: 120.20 qps: 2399.28 (r/w/o: 1677.05/481.82/240.41) lat (ms,95%): 977.74 err/s: 0.00 reconn/s: 0.00
[ 45s ] thds: 64 tps: 131.59 qps: 2631.87 (r/w/o: 1843.11/525.57/263.19) lat (ms,95%): 846.57 err/s: 0.00 reconn/s: 0.00
[ 50s ] thds: 64 tps: 130.00 qps: 2591.45 (r/w/o: 1814.03/517.41/260.00) lat (ms,95%): 1013.60 err/s: 0.00 reconn/s: 0.00
[ 55s ] thds: 64 tps: 123.60 qps: 2496.79 (r/w/o: 1745.19/504.20/247.40) lat (ms,95%): 861.95 err/s: 0.00 reconn/s: 0.00
[ 60s ] thds: 64 tps: 129.00 qps: 2557.67 (r/w/o: 1791.25/508.41/258.01) lat (ms,95%): 909.80 err/s: 0.00 reconn/s: 0.00
SQL statistics:
    queries performed:
        read:                            211694
        write:                           60482
        other:                           30244
        total:                           302420
    transactions:                        15121  (249.37 per sec.)
    queries:                             302420 (4987.38 per sec.)
    ignored errors:                      0      (0.00 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          60.6353s
    total number of events:              15121

Latency (ms):
         min:                                    4.91
         avg:                                  255.04
         max:                                 2008.54
         95th percentile:                      733.00
         sum:                              3856511.86

Threads fairness:
    events (avg/stddev):           236.2656/7.64
    execution time (avg/stddev):   60.2580/0.13

</pre>