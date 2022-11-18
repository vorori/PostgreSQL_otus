
# 1
Настройте выполнение контрольной точки раз в 30 секунд.

sudo -u postgres psql -c "ALTER SYSTEM SET checkpoint_timeout = '30s';"
sudo systemctl restart postgresql-14
sudo -u postgres psql -c "SHOW checkpoint_timeout;"


# 2
10 минут c помощью утилиты pgbench подавайте нагрузку.

sudo -u postgres psql -c "select pg_stat_reset();"
sudo -u postgres psql -c "select pg_stat_reset_shared('bgwriter');"


объем wal файлов
sudo du -h /var/lib/pgsql/14/data/pg_wal/ # 16mb

/usr/pgsql-14/bin/pgbench -i postgres -U postgres
/usr/pgsql-14/bin/pgbench -c8 -P 60 -T 600 -U postgres postgres


# 3
Измерьте, какой объем журнальных файлов был сгенерирован за это время. Оцените, какой объем приходится в среднем на одну контрольную точку.

при помощи этого запроса определили список контрольных точек и среднее время выполнения

<pre>
SELECT
total_checkpoints,
seconds_since_start / total_checkpoints / 60 AS minutes_between_checkpoints
FROM
(SELECT
EXTRACT(EPOCH FROM (now() - pg_postmaster_start_time())) AS seconds_since_start,
(checkpoints_timed+checkpoints_req) AS total_checkpoints
FROM pg_stat_bgwriter
) AS sub;
</pre>





объем wal файлов
sudo du -h /var/lib/pgsql/14/data/pg_wal/ # 64mb


# 4
Проверьте данные статистики: все ли контрольные точки выполнялись точно по расписанию. Почему так произошло?

статистика выполнения контрольных точек
<pre>
total_checkpoints | minutes_between_checkpoints
-------------------+-----------------------------
                22 |      0.52831149166666666667
(1 row)
</pre>


Процесс контрольной точки сервера время от времени автоматически выполняет контрольную точку. Контрольная точка запускается каждые секунды checkpoint_timeout или если max_wal_size вот-вот будет превышен, в зависимости от того, что наступит раньше. Настройки по умолчанию — 5 минут и 1 ГБ соответственно. Если после предыдущей контрольной точки не было записано ни одного WAL, новые контрольные точки будут пропущены, даже если checkpoint_timeoutони были пройдены.Также можно принудительно установить контрольную точку. с помощью команды SQL CHECKPOINT.

Уменьшение checkpoint_timeoutи/или max_wal_size приведет к увеличению количества контрольных точек


<pre>
посмотрим на pg_stat_bgwriter
sudo -u postgres psql -c "SELECT * FROM pg_stat_bgwriter \gx"
</pre>

<pre>
-[ RECORD 1 ]---------+------------------------------
checkpoints_timed     | 23
checkpoints_req       | 0
checkpoint_write_time | 618784
checkpoint_sync_time  | 739
buffers_checkpoint    | 42888
buffers_clean         | 0
maxwritten_clean      | 0
buffers_backend       | 1949
buffers_backend_fsync | 0
buffers_alloc         | 2282
stats_reset           | 2022-11-18 19:30:01.609299+00
</pre>

<pre>
Здесь, в числе прочего, мы видим количество выполненных контрольных точек:

checkpoints_timed — по расписанию (по достижению checkpoint_timeout)
checkpoints_req — по требованию (в том числе по достижению max_wal_size).
</pre>

# 5
Сравните tps в синхронном/асинхронном режиме утилитой pgbench. Объясните полученный результат.

/usr/pgsql-14/bin/pgbench -c8 -P 10 -T 50 -U postgres postgres

<pre>
progress: 10.0 s, 543.7 tps, lat 14.674 ms stddev 9.418
progress: 20.0 s, 669.9 tps, lat 11.936 ms stddev 6.640
progress: 30.0 s, 408.0 tps, lat 19.528 ms stddev 18.402
progress: 40.0 s, 587.7 tps, lat 13.661 ms stddev 13.060
progress: 50.0 s, 544.9 tps, lat 14.674 ms stddev 10.820
</pre>



sudo -u postgres psql -c "ALTER SYSTEM SET synchronous_commit = 'off';"


sudo systemctl restart postgresql-14

sudo -u postgres psql -c "SHOW synchronous_commit;"

/usr/pgsql-14/bin/pgbench -c8 -P 10 -T 50 -U postgres postgres

<pre>
starting vacuum...end.
progress: 10.0 s, 3881.9 tps, lat 2.052 ms stddev 0.785
progress: 20.0 s, 3918.5 tps, lat 2.036 ms stddev 0.802
progress: 30.0 s, 3939.2 tps, lat 2.025 ms stddev 0.770
progress: 40.0 s, 3699.9 tps, lat 2.156 ms stddev 0.901
progress: 50.0 s, 3642.7 tps, lat 2.189 ms stddev 0.931
</pre>


<pre>
c clients --client=clients Количество смоделированных клиентов, то есть количество одновременных сеансов базы данных. По умолчанию 1.

-P sec --progress=sec Показывать отчет о проделанной работе каждую sec секунду. Отчет включает время с начала выполнения, TPS с момента последнего отчета, а также среднюю задержку транзакции и стандартное отклонение с момента последнего отчета. При регулировании ( -R) задержка вычисляется относительно запланированного времени начала транзакции, а не фактического времени начала транзакции, поэтому она также включает среднее время задержки расписания.

-T seconds --time=seconds Запустите тест на это количество секунд, а не на фиксированное количество транзакций на клиента. -t и -Tявляются взаимоисключающими.
</pre>


в асинхронном режиме postgres работает намного быстрее

Разница почти в 6 раза.

более эффективно сбрасываем данные на диск, теперь за вместо того что бы писать на диск на каждое действие в кластере, мы делаем это отдельным процессом, по расписанию и пачками.


# 6
Создайте новый кластер с включенной контрольной суммой страниц. Создайте таблицу. Вставьте несколько значений. Выключите кластер. Измените пару байт в таблице. Включите 
кластер и сделайте выборку из таблицы. Что и почему произошло? как проигнорировать ошибку и продолжить работу?


<pre>
su - postgres

/usr/pgsql-14/bin/initdb -D /var/lib/pgsql/14/data -k 

/usr/pgsql-14/bin/pg_ctl -D /var/lib/pgsql/14/data -l logfile start
</pre>



<pre>
su - postgres -c 'psql -c "SHOW data_checksums;"'

postgres=# SHOW data_checksums;
 data_checksums
----------------
 on
(1 row)
</pre>


<pre>
create table test(c1 text);

insert into test (c1) values ('YO');

insert into test (c1) values ('KY');

select * from test;

SELECT pg_relation_filepath('test'); # base/14486/16384

/usr/pgsql-14/bin/pg_ctl -D /var/lib/pgsql/14/data -l logfile stop

Что и почему произошло?

WARNING:  page verification failed, calculated checksum 25708 but expected 61710
ERROR:  invalid page in block 0 of relation base/14486/16384

data checksums гарантирует целостность данных на уровне байтов в файлах и ругается о том что файл был поврежден
</pre>



Как проигнорировать ошибку и продолжить работу?

<pre>
Есть сразу несколько вариантов:

в самом сеансе можно попросить postgres игнорировать ошибку и выдывать что есть
можно найти поврежденные строки и удалить их
можно выставить настройку зануляющую поврежденные строки и выполнить полный вакуум


SET zero_damaged_pages = on;
vacuum full test;
select * from messages;
</pre>