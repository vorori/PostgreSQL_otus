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

#### Протестировать pg_bench

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

#### 3)

#### Выставить оптимальные настройки

<pre>

</pre>

#### 4)

####  Проверить насколько выросла производительность


<pre>

</pre>


#### 5)

####  Настроить кластер на оптимальную производительность не обращая внимания на стабильность БД


<pre>

</pre>

