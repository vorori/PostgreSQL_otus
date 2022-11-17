# 1
создать GCE инстанс типа e2-medium и диском 10GB

создал centos7 sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm

VM 2 CPU 

4 ОЗУ

10 GB HDD


# 2
установить на него PostgreSQL 14 с дефолтными настройками

sudo yum install postgresql14-contrib 

sudo /usr/pgsql-14/bin/postgresql-14-setup initdb 

sudo systemctl enable --now postgresql-14 

systemctl status postgresql-14
systemctl restart postgresql-14

# 3
применить параметры настройки PostgreSQL из прикрепленного к материалам занятия файла

в настройках которые приложены к заданию есть параметр max_wal_size = 16GB который никак не вписывается в наши параметры сервера у нас та на сервере 10gb hdd соответственно он нам может испортить всю работу кластера

<pre>
max_connections = 40
shared_buffers = 1GB
effective_cache_size = 3GB
maintenance_work_mem = 512MB
checkpoint_completion_target = 0.9
wal_buffers = 16MB
default_statistics_target = 500
random_page_cost = 4
effective_io_concurrency = 2
work_mem = 6553kB
min_wal_size = 4GB
max_wal_size = 16GB
</pre>

настройки которые я применил

<pre>
ALTER SYSTEM SET max_connections = '40'; 
ALTER SYSTEM SET shared_buffers = '1GB';
ALTER SYSTEM SET effective_cache_size = '3GB';
ALTER SYSTEM SET maintenance_work_mem = '512MB';
ALTER SYSTEM SET checkpoint_completion_target = '0.9';
ALTER SYSTEM SET wal_buffers = '16MB';
ALTER SYSTEM SET default_statistics_target = '500';
ALTER SYSTEM SET random_page_cost = '4';
ALTER SYSTEM SET effective_io_concurrency = '2';
ALTER SYSTEM SET work_mem = '6553kB';
ALTER SYSTEM SET min_wal_size = '2GB';
ALTER SYSTEM SET max_wal_size = '4GB';
</pre>

# 4
выполнить pgbench -i postgres

Инициализация тестовой базы

/usr/pgsql-14/bin/pgbench -i postgres -U postgres

<pre>
dropping old tables...
creating tables...
generating data (client-side)...
100000 of 100000 tuples (100%) done (elapsed 0.01 s, remaining 0.00 s)
vacuuming...
creating primary keys...
done in 0.34 s (drop tables 0.01 s, create tables 0.01 s, client-side generate 0.16 s, vacuum 0.07 s, primary keys 0.08 s).
</pre>



этим скриптом проверяем количесто мертвых строк и автовакумма

sudo -u postgres psql -c "select relname, n_live_tup, n_dead_tup, autovacuum_count from pg_stat_all_tables where relname like ('pgbench_%');"

<pre>
 relname      | n_live_tup | n_dead_tup | autovacuum_count | 
------------------+------------+------------+------------------ 
 pgbench_history  |          0 |          0 |                0 
 pgbench_tellers  |         10 |          0 |                0 
 pgbench_accounts |     100000 |          0 |                0 
 pgbench_branches |          1 |          0 |                0 
(4 rows)  
</pre>

# 5
запустить pgbench -c8 -P 60 -T 600 -U postgres postgres


описание ключей:

c clients
--client=clients Количество смоделированных клиентов, то есть количество одновременных сеансов базы данных. По умолчанию 1.

-P sec
--progress=sec
Показывать отчет о проделанной работе каждую sec секунду. Отчет включает время с начала выполнения, TPS с момента последнего отчета, а также среднюю задержку транзакции и стандартное отклонение с момента последнего отчета. При регулировании ( -R) задержка вычисляется относительно запланированного времени начала транзакции, а не фактического времени начала транзакции, поэтому она также включает среднее время задержки расписания.

-T seconds
--time=seconds
Запустите тест на это количество секунд, а не на фиксированное количество транзакций на клиента. -t и -Tявляются взаимоисключающими.


И собственно запуск самого теста

/usr/pgsql-14/bin/pgbench -c8 -P 60 -T 600 -U postgres postgres > /tmp/log.txt 2>&1


# 6
дать отработать до конца

данные после выполнения pgbench

cat /tmp/log.txt

pgbench (14.6)\
starting vacuum...end.\
progress: 60.0 s, 483.1 tps, lat 16.546 ms stddev 15.408\
progress: 120.0 s, 429.6 tps, lat 18.615 ms stddev 17.830\
progress: 180.0 s, 389.4 tps, lat 20.531 ms stddev 19.988\
progress: 240.0 s, 587.0 tps, lat 13.631 ms stddev 10.522\
progress: 300.0 s, 569.5 tps, lat 14.039 ms stddev 10.560\
progress: 360.0 s, 399.1 tps, lat 20.040 ms stddev 17.643\
progress: 420.0 s, 522.7 tps, lat 15.302 ms stddev 14.202\
progress: 480.0 s, 513.8 tps, lat 15.564 ms stddev 13.265\
progress: 540.0 s, 581.3 tps, lat 13.757 ms stddev 13.175\
progress: 600.0 s, 476.2 tps, lat 16.792 ms stddev 16.420\
transaction type: <builtin: TPC-B (sort of)>\
scaling factor: 1\
query mode: simple\
number of clients: 8\
number of threads: 1\
duration: 600 s\
number of transactions actually processed: 297116\
latency average = 16.150 ms\
latency stddev = 14.981 ms\
initial connection time = 15.528 ms\
tps = 495.179260 (without initial connection time)


График tps по получившимся значениям

![](https://quickchart.io/chart?c={%20type:%20%27bar%27,%20data:%20{%20labels:%20[%271%27,%272%27,%273%27,%274%27,%275%27,%276%27,%277%27,%278%27,%279%27,%2710%27],%20datasets:%20[{%20label:%20%27tps%27,%20data:%20[%20,483.1,429.6,389.4,587.0,569.5,399.1,522.7,513.8,581.3,476.2%20]%20}]%20}})


снова проверяем статистику после первого прохода pgbench

sudo -u postgres psql -c "select relname, n_live_tup, n_dead_tup, autovacuum_count from pg_stat_all_tables where relname like ('pgbench_%');"

<pre>
 relname      | n_live_tup | n_dead_tup | autovacuum_count\
------------------+------------+------------+------------------\
 pgbench_history  |     297101 |          0 |                7\
 pgbench_tellers  |         10 |          0 |               11\
 pgbench_accounts |     100000 |       7138 |                1\
 pgbench_branches |          1 |          0 |               11

</pre>
# 7
дальше настроить autovacuum максимально эффективно

смотрим текушие настройки Autovacuum

sudo -u postgres psql -c "SELECT name, setting FROM pg_settings WHERE category like '%Autovacuum%';"

<pre>
name                  |  setting\
---------------------------------------+-----------
 autovacuum                            | on
 autovacuum_analyze_scale_factor       | 0.1
 autovacuum_analyze_threshold          | 50
 autovacuum_freeze_max_age             | 200000000
 autovacuum_max_workers                | 3
 autovacuum_multixact_freeze_max_age   | 400000000
 autovacuum_naptime                    | 60
 autovacuum_vacuum_cost_delay          | 2
 autovacuum_vacuum_cost_limit          | -1
 autovacuum_vacuum_insert_scale_factor | 0.2
 autovacuum_vacuum_insert_threshold    | 1000
 autovacuum_vacuum_scale_factor        | 0.2
 autovacuum_vacuum_threshold           | 50
</pre>

# меняю настройки Autovacuum

#моя конфигурация автовакумма:
0)
maintenance_work_mem = 256GB # я установил в это Память для работы команды VACUUM Для этого в локальной памяти процесса VACUUM выделяется фрагмент размером maintenance_work_mem (или меньше, если таблица небольшая).  Отметим, что это память выделяется сразу в полном объеме, а не по мере необходимости. И эта память сразу выделяется каждому процессу автовакума если у нас 10 процессов автовакума то 10 * 256 соответственно израсходуется.

ВАЖНО 

НАСТРОЙКИ ВАКУМА ДЛЯ РУЧНОГО ЗАПУСКА не трогал
команда VACUUM ANALYZE Не конфликтует с обычной активностью в системе. Если запустим VACUUM ANALYZE то выполнится и отчистка и пересчет статистики.

<pre>
VACUUM ANALYZE TABLE_NAME;
</pre>

ВАЖНО 

ЗАПУСК ВАКУМА  FULL
ТО БУДУТ ВЫПОЛНЯТЬСЯ ЭКСКЛЮЗИВНЫЕ БЛОКИРОВКИ ПОТОМУ ЧТО БУДЕТ ВЫПОЛНЕНО ФИЗИЧЕСКОЕ ПРЕСТРОЕНИЕ ДАННЫХ 
И ОЧЕНЬВАЖНО ПОМНИТЬ ЧТО ЕСЛИ МЫ ЗАПУСКАЕМ VACUUM FULL ТО НАМ ПОТРЕБУЕТСЯ ДОПОЛНИТЕЛЬНОЕ ПРОСТРАНСТВО НА ДИСКЕ ЧТОБЫ ВЫПОЛНИТЬ ЭТУ ОПЕРАЦИЮ ПРЕДПОЛОЖИМ ТАБЛИЦА ЗАНИМАЕТ 10GB ЧТОБЫ ЕЕ ПРЕСТРОИТЬ НАМ ПОТРЕБУЕТСЯ ЕШЕ 10 GB!!! Для нашего экспремента обьем таблиц используется намного меньше.

<pre>
VACUUM FULL TABLE_NAME;
</pre>


ВАЖНО НАСТРОЙКИ ВАКУМА ДЛЯ АВТОМАТИЧЕСКОГО ЗАПУСКА

autovacuum_vacuum_scale_factor = 0.05  
я установил в 0.05 # (знач. по умолчанию 20%(мертвых строк) объяснение: как только у нас появится 20% мертвых-измененных строк плюс кол-во + знач. autovacuum_vacuum_threshold которое равно 50 строками то есть если строки изменятся в количестве более суммы этих двух параметров то запускается авто отчистка таблицы. Считается такое мнение, что значение 0.2 это 20% это очень большая цифра это значение #нужно уменьшать в разы примерно 10 % или 5% или 1% потому как таблицы бывают разные и в каких то таблицах количество строк может быть больше миллионов и тогда нужно рассмотреть возможность установить эти значения индивидуально применимые к конкретной таблице.

autovacuum_analyze_scale_factor = 0.02  я установил в 0.02 
Настройки автоанализа  ( 10% вполне возможно его уменьшить   значение 0.02 это означает 2 процента)

autovacuum_vacuum_cost_delay = 2ms  # (на какое время нужно уснуть и снова начать работу вакккум осавил)

autovacuum_max_workers = 3 (оставил 3)
в нашем случае вероятно не имеет смысла больше 4, т.к. pgbench создал столько табличек, выше будет просто нечего чистить (плюс не стоить забывать что на нашей виртуалке всего 2 ядра)

autovacuum_vacuum_threshold (оставил 50 по умолчанию)
Указывает минимальное количество обновленных или удаленных кортежей, необходимое для запуска a VACUUMв любой таблице. По умолчанию 50 кортежей. Этот параметр можно задать только в postgresql.confфайле или в командной строке сервера; но этот параметр можно переопределить для отдельных таблиц, изменив параметры хранения таблиц.


настройки которые применил 
<pre>
ALTER SYSTEM SET autovacuum_vacuum_scale_factor = '0.05'; 
ALTER SYSTEM SET autovacuum_analyze_scale_factor = '0.02';
ALTER SYSTEM SET autovacuum_vacuum_cost_delay = '2';
ALTER SYSTEM SET autovacuum_max_workers = '3';
ALTER SYSTEM SET autovacuum_vacuum_threshold = '50';
</pre>

# 8
построить график по получившимся значениям так чтобы получить максимально ровное значение tps

/usr/pgsql-14/bin/pgbench -c8 -P 60 -T 600 -U postgres postgres > /tmp/newlog.txt 2>&1


<pre>
starting vacuum...end.
progress: 60.0 s, 548.3 tps, lat 14.574 ms stddev 12.076
progress: 120.0 s, 526.2 tps, lat 15.199 ms stddev 13.026
progress: 180.0 s, 476.8 tps, lat 16.772 ms stddev 15.007
progress: 240.0 s, 577.2 tps, lat 13.849 ms stddev 10.914
progress: 300.0 s, 580.7 tps, lat 13.774 ms stddev 10.296
progress: 360.0 s, 579.0 tps, lat 13.809 ms stddev 10.300
progress: 420.0 s, 374.2 tps, lat 21.368 ms stddev 19.222
progress: 480.0 s, 427.7 tps, lat 18.697 ms stddev 15.241
progress: 540.0 s, 430.2 tps, lat 18.591 ms stddev 16.224
progress: 600.0 s, 477.4 tps, lat 16.964 ms stddev 13.861
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
duration: 600 s
number of transactions actually processed: 299524
latency average = 16.019 ms
latency stddev = 13.701 ms
initial connection time = 19.258 ms
tps = 499.206700 (without initial connection time)
</pre>


скриптом формируем график
<pre>
DATAnew=$(cat /tmp/newlog.txt | head -n -11 | tail -n +2 | cut -d " " -f 4 | awk -vORS=, '{ print $1 }' | sed 's/,$/\n/')
echo "https://quickchart.io/chart?c={ type: 'bar', data: { labels: ['1','2','3','4','5','6','7','8','9','10'], datasets: [{ label: 'tps_new', data: [ $DATAnew ] }] }}"
</pre>

![](https://quickchart.io/chart?c={%20type:%20%27bar%27,%20data:%20{%20labels:%20[%271%27,%272%27,%273%27,%274%27,%275%27,%276%27,%277%27,%278%27,%279%27,%2710%27],%20datasets:%20[{%20label:%20%27tps_new%27,%20data:%20[%20,548.3,526.2,476.8,577.2,580.7,579.0,374.2,427.7,430.2,471.4%20]%20}]%20}})




# статистика tps значения по умолчанию autovacuum 
<pre>
latency average = 16.150 ms
latency stddev = 14.981 ms
initial connection time = 15.528 ms
tps = 495.179260 (without initial connection time)
</pre>

# статистика tps после настройки autovacuum  
<pre>
latency average = 16.019 ms 
latency stddev = 13.701 ms 
initial connection time = 19.258 ms 
tps = 499.206700 (without initial connection time)
</pre>

в целом получили небольшой прирост по tps 