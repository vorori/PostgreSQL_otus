
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

[ 60s ] thds: 64 tps: 189.96 qps: 3809.35 (r/w/o: 2668.07/760.26/381.02) lat (ms,95%): 943.16 err/s: 0.00 reconn/s: 0.00
[ 120s ] thds: 64 tps: 213.05 qps: 4264.67 (r/w/o: 2985.06/853.37/426.25) lat (ms,95%): 846.57 err/s: 0.02 reconn/s: 0.00
[ 180s ] thds: 64 tps: 315.63 qps: 6318.40 (r/w/o: 4422.69/1264.27/631.45) lat (ms,95%): 601.29 err/s: 0.07 reconn/s: 0.00
[ 240s ] thds: 64 tps: 184.93 qps: 3696.77 (r/w/o: 2588.12/738.70/369.95) lat (ms,95%): 877.61 err/s: 0.03 reconn/s: 0.00
[ 300s ] thds: 64 tps: 228.13 qps: 4566.60 (r/w/o: 3196.40/913.85/456.35) lat (ms,95%): 816.63 err/s: 0.03 reconn/s: 0.00
[ 360s ] thds: 64 tps: 180.52 qps: 3606.66 (r/w/o: 2525.01/720.53/361.12) lat (ms,95%): 943.16 err/s: 0.00 reconn/s: 0.00
[ 420s ] thds: 64 tps: 234.36 qps: 4691.43 (r/w/o: 3283.87/938.75/468.82) lat (ms,95%): 787.74 err/s: 0.03 reconn/s: 0.00
[ 480s ] thds: 64 tps: 134.44 qps: 2685.65 (r/w/o: 1880.35/536.42/268.88) lat (ms,95%): 1050.76 err/s: 0.00 reconn/s: 0.00
[ 540s ] thds: 64 tps: 150.50 qps: 3010.93 (r/w/o: 2107.10/602.73/301.10) lat (ms,95%): 1032.01 err/s: 0.02 reconn/s: 0.00
[ 600s ] thds: 64 tps: 228.80 qps: 4574.78 (r/w/o: 3202.66/914.33/457.78) lat (ms,95%): 733.00 err/s: 0.05 reconn/s: 0.00
SQL statistics:
    queries performed:
        read:                            1731842
        write:                           494752
        other:                           247436
        total:                           2474030
    transactions:                        123688 (205.16 per sec.)
    queries:                             2474030 (4103.55 per sec.)
    ignored errors:                      15     (0.02 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          602.8977s
    total number of events:              123688

Latency (ms):
         min:                                    3.92
         avg:                                  310.74
         max:                                 4596.30
         95th percentile:                      861.95
         sum:                             38434523.53

Threads fairness:
    events (avg/stddev):           1932.6250/21.10
    execution time (avg/stddev):   600.5394/0.24


</pre>



# 4
настроить кластер PostgreSQL 14 на максимальную производительность не
обращая внимание на возможные проблемы с надежностью в случае
аварийной перезагрузки виртуальной машины

#применил вот такие настройки

ALTER SYSTEM SET
 max_connections = '98';
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
ALTER SYSTEM SET
 max_worker_processes = '4';
ALTER SYSTEM SET
 max_parallel_workers_per_gather = '2';
ALTER SYSTEM SET
 max_parallel_workers = '4';
ALTER SYSTEM SET
 max_parallel_maintenance_workers = '2';









# 5
нагрузить кластер через утилиту
https://github.com/Percona-Lab/sysbench-tpcc (требует установки
https://github.com/akopytov/sysbench) или через утилиту pgbench (https://postgrespro.ru/docs/postgrespro/14/pgbench)
написать какого значения tps удалось достичь, показать какие параметры в
какие значения устанавливали и почему


с новыми настройками получили прирост в tps по сравнению с первым тестом на настройках по умолчанию

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