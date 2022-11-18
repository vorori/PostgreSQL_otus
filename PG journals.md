
# 1
Настройте выполнение контрольной точки раз в 30 секунд.

ALTER SYSTEM SET checkpoint_timeout = '30s';
sudo systemctl restart postgresql-14

# 2
10 минут c помощью утилиты pgbench подавайте нагрузку.

sudo -u postgres psql -c "select pg_stat_reset()"
sudo -u postgres psql -c "select pg_stat_reset_shared('bgwriter')"


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


<pre>
 total_checkpoints | minutes_between_checkpoints
-------------------+-----------------------------
                35 |      0.61807570095238095167
(1 row)
</pre>


объем wal файлов
sudo du -h /var/lib/pgsql/14/data/pg_wal/ # 160mb


# 4
Проверьте данные статистики: все ли контрольные точки выполнялись точно по расписанию. Почему так произошло?





# 5
Сравните tps в синхронном/асинхронном режиме утилитой pgbench. Объясните полученный результат.


# 6
Создайте новый кластер с включенной контрольной суммой страниц. Создайте таблицу. Вставьте несколько значений. Выключите кластер. Измените пару байт в таблице. Включите 


# 7
кластер и сделайте выборку из таблицы. Что и почему произошло? как проигнорировать ошибку и продолжить работу?