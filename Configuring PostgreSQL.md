
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
настроить кластер PostgreSQL 14 на максимальную производительность не
обращая внимание на возможные проблемы с надежностью в случае
аварийной перезагрузки виртуальной машины


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
нагрузить кластер через утилиту
https://github.com/Percona-Lab/sysbench-tpcc (требует установки
https://github.com/akopytov/sysbench) или через утилиту pgbench (https://postgrespro.ru/docs/postgrespro/14/pgbench)
написать какого значения tps удалось достичь, показать какие параметры в
какие значения устанавливали и почему


<pre>
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
</pre>