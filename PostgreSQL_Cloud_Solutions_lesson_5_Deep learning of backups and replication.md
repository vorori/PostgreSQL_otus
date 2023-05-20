#### 1)

#### Делам бэкап Постгреса используя WAL-G или pg_probackup и восстанавливаемся на другом кластере



#### 1.1) Создаем два кластера
<pre>
создал 2 vm centos core 2 озу 4gb 
otus_vm1_centos ip 192.168.2.4
otus_vm2_centos ip 192.168.2.8


sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
sudo yum -y repolist
sudo yum install postgresql15-contrib

sudo /usr/pgsql-15/bin/postgresql-15-setup initdb

systemctl enable postgresql-15.service
systemctl status postgresql-15.service
systemctl restart postgresql-15.service

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

systemctl restart postgresql-15.service
</pre>


#### 1.2) подготовительные мероприятия для настройки асинхронной репликации for linux
<pre> 
#otus_vm1_centos ip 192.168.2.4
firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.2.8" port protocol="tcp" port="5432" accept"


#otus_vm2_centos ip 192.168.2.8
firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.2.4" port protocol="tcp" port="5432" accept"

firewall-cmd --reload		
firewall-cmd --list-all
</pre>


#### 1.3) подготовительные мероприятия для настройки асинхронной репликации for postgres

<pre> 
sudo -u postgres psql -c "ALTER SYSTEM SET wal_level = 'replica';"
sudo -u postgres psql -c "ALTER SYSTEM SET listen_addresses TO '*';"
sudo -u postgres psql -c "ALTER SYSTEM SET synchronous_commit TO 'on';"
systemctl restart postgresql-15.service
</pre>

###Теперь включаем на VM1 (он же будет мастером режим асинхронной репликации)
###создаю на VM1 мастере пользователя который будет отвечать за репликацию 

<pre> 
sudo -u postgres psql -c "CREATE ROLE user_replicator WITH REPLICATION PASSWORD 'klJlkdfsjImdsksdmsd98' LOGIN;"
</pre>


###доступ для роли user_replicator
<pre> 
vim /var/lib/pgsql/15/data/pg_hba.conf
host    replication     user_replicator          192.168.2.8/32            scram-sha-256
</pre>


####создаю слот на мастере VM1
<pre> 
sudo -u postgres psql -c "SELECT * FROM pg_create_physical_replication_slot('my_slot_replication');"
</pre>

###проверяю слот
<pre> 
sudo -u postgres 
psql
SELECT * FROM pg_replication_slots; \gx

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


#на VM2 останавливаю кластер удаляю данные 
<pre> 
sudo systemctl stop postgresql-15
sudo systemctl status postgresql-15
sudo -u postgres rm -rf /var/lib/pgsql/15/data/*
sudo -u postgres pg_basebackup -h 192.168.2.4 -D /var/lib/pgsql/15/data -U user_replicator -P -v -R -X stream -S my_slot_replication
</pre>

#проверяю что появилась дополнителная информация по слоту репликации
<pre> 
cat /var/lib/pgsql/15/data/postgresql.auto.conf
</pre>

#запускаю кластер на VM2
<pre> 
sudo systemctl restart postgresql-15
</pre>

#Сейчас VM2 сервер является репликой (находится в режиме восстановления):
<pre> 
sudo -u postgres psql -c "SELECT pg_is_in_recovery();"

pg_is_in_recovery
-------------------
 t
(1 row)
</pre>



#на главном сервере VM1 проверяем репликацию
<pre> 
postgres=# SELECT * FROM pg_stat_replication \gx
-[ RECORD 1 ]----+------------------------------
pid              | 3360
usesysid         | 16388
usename          | user_replicator
application_name | walreceiver
client_addr      | 192.168.2.8
client_hostname  |
client_port      | 58390
backend_start    | 2023-05-20 23:02:36.681187+03
backend_xmin     |
state            | streaming
sent_lsn         | 0/3000148
write_lsn        | 0/3000148
flush_lsn        | 0/3000148
replay_lsn       | 0/3000148
write_lag        |
flush_lag        |
replay_lag       |
sync_priority    | 0
sync_state       | async
reply_time       | 2023-05-20 23:05:50.380522+03

все ОК!
</pre>



#### 1.4) install WAL-G
-- sudo rm /usr/local/bin/wal-g
wget https://github.com/philyuchkoff/wal-g-centos7/releases/download/2.0.1/wal-g-pg && mkdir /usr/local/bin/wal-g && sudo mv wal-g-pg /usr/local/bin/wal-g
sudo ls -l /usr/local/bin/wal-g

sudo rm -rf /var/postgre_backup && sudo mkdir /var/postgre_backup && sudo chmod 777 /var/postgre_backup

chown postgres:postgres /var/postgre_backup

#### 1.5)Создаем файл конфигурации для wal-g
sudo su - postgres
vim ~/.walg.json


-- https://github.com/wal-g/wal-g/blob/master/docs/PostgreSQL.md
-- https://github.com/wal-g/wal-g/blob/master/docs/STORAGES.md

{
    "WALG_FILE_PREFIX": "/var/postgre_backup",
    "WALG_COMPRESSION_METHOD": "brotli",
    "WALG_DELTA_MAX_STEPS": "5",
    "PGDATA": "/var/lib/pgsql/15/data",
    "PGHOST": "/var/run/postgresql/.s.PGSQL.5432"
}

-- опция для дебага
--     "WALG_LOG_LEVEL": "DEVEL"

mkdir /var/lib/pgsql/15/data/log_wal_g
chown postgres:postgres /var/lib/pgsql/15/data/log_wal_g
chmod 700 /var/lib/pgsql/15/data/log_wal_g

vim /var/lib/pgsql/15/data/postgresql.conf
или
echo "wal_level=replica" >> /var/lib/pgsql/15/data/postgresql.auto.conf
echo "archive_mode=on" >> /var/lib/pgsql/15/data/postgresql.auto.conf
echo "archive_command='/usr/local/bin/wal-g/wal-g-pg wal-push \"%p\" >> /var/lib/pgsql/15/data/log_wal_g/archive_command.log 2>&1' " >> /var/lib/pgsql/15/data/postgresql.auto.conf
echo "archive_timeout=60" >> /var/lib/pgsql/15/data/postgresql.auto.conf 
echo "restore_command='/usr/local/bin/wal-g/wal-g-pg wal-fetch \"%f\" \"%p\" >> /var/lib/pgsql/15/data/log_wal_g/restore_command.log 2>&1' " >> /var/lib/pgsql/15/data/postgresql.auto.conf

sudo systemctl restart postgresql-15
sudo systemctl status postgresql-15



#### 1.5)наполняем данными 
su - postgres

-- Создадим новую базу данных
psql -c "CREATE DATABASE otus;"

-- Таблицу в этой базе данных и заполним ее тестовыми данными
psql otus -c "create table test(i int);"
psql otus -c "insert into test values (10), (20), (30);"
psql otus -c "select * from test;"


#### 1.6) выполняем бекап
выполнил бэкап
su - postgres
su - postgres
su - postgres

-bash-4.2$ /usr/local/bin/wal-g/wal-g-pg backup-push /var/lib/pgsql/15/data
-bash-4.2$ /usr/local/bin/wal-g/wal-g-pg backup-push /var/lib/pgsql/15/data
-bash-4.2$ /usr/local/bin/wal-g/wal-g-pg backup-push /var/lib/pgsql/15/data


#### 1.7) смотрим на наличие ошибок 
cat /var/lib/pgsql/15/data/log/postgresql-Sun.log
cat /var/lib/pgsql/15/data/log_wal_g/archive_command.log

-bash-4.2$ cat /var/lib/pgsql/15/data/log_wal_g/archive_command.log
INFO: 2023/05/21 00:29:08.700980 FILE PATH: 00000001000000000000000C.br
INFO: 2023/05/21 00:30:05.936070 FILE PATH: 00000001000000000000000D.br
INFO: 2023/05/21 00:30:06.353350 FILE PATH: 00000001000000000000000E.00000060.backup.br
INFO: 2023/05/21 00:30:06.361288 FILE PATH: 00000001000000000000000E.br
INFO: 2023/05/21 00:30:40.390422 FILE PATH: 00000001000000000000000F.br
INFO: 2023/05/21 00:30:56.695581 FILE PATH: 000000010000000000000010.br
INFO: 2023/05/21 00:30:56.824121 FILE PATH: 000000010000000000000011.00000028.backup.br
INFO: 2023/05/21 00:30:56.842308 FILE PATH: 000000010000000000000011.br


#### 1.8) смотрим какие бекапы есть в наличии
/usr/local/bin/wal-g/wal-g-pg backup-list
/usr/local/bin/wal-g/wal-g-pg backup-list
/usr/local/bin/wal-g/wal-g-pg backup-list

-bash-4.2$ /usr/local/bin/wal-g/wal-g-pg backup-list
name                                                     modified                  wal_segment_backup_start
base_00000001000000000000000E                            2023-05-21T00:30:06+03:00 00000001000000000000000E
base_000000010000000000000011_D_00000001000000000000000E 2023-05-21T00:30:56+03:00 000000010000000000000011
base_000000010000000000000013_D_000000010000000000000011 2023-05-21T00:32:42+03:00 000000010000000000000013


#### 1.9) обновляем наши данные
psql otus -c "UPDATE test SET i = 3 WHERE i = 30"
psql otus -c "select * from test;"


#### 1.10) делаем еше один бекап
-- make delta
/usr/local/bin/wal-g/wal-g-pg backup-push /var/lib/pgsql/15/data


#### 1.10) смотрим снова на лист бекапов
-bash-4.2$ /usr/local/bin/wal-g/wal-g-pg backup-list
name                                                     modified                  wal_segment_backup_start
base_00000001000000000000000E                            2023-05-21T00:30:06+03:00 00000001000000000000000E
base_000000010000000000000011_D_00000001000000000000000E 2023-05-21T00:30:56+03:00 000000010000000000000011
base_000000010000000000000013_D_000000010000000000000011 2023-05-21T00:32:42+03:00 000000010000000000000013
base_000000010000000000000016_D_000000010000000000000013 2023-05-21T00:39:49+03:00 000000010000000000000016


#### 1.11) бекапы физически лежат сдесь
cd /var/postgre_backup
-bash-4.2$ ls -l
total 8
drwxr-xr-x. 6 postgres postgres 4096 May 21 00:39 basebackups_005
drwx------. 2 postgres postgres 4096 May 21 00:39 wal_005
cd basebackups_005
-bash-4.2$ cd basebackups_005
-bash-4.2$ ls -l
total 16
drwxr-xr-x. 3 postgres postgres  76 May 21 00:30 base_00000001000000000000000E
-rw-r--r--. 1 postgres postgres 379 May 21 00:30 base_00000001000000000000000E_backup_stop_sentinel.json
drwxr-xr-x. 3 postgres postgres  76 May 21 00:30 base_000000010000000000000011_D_00000001000000000000000E
-rw-r--r--. 1 postgres postgres 503 May 21 00:30 base_000000010000000000000011_D_00000001000000000000000E_backup_stop_sentinel.json
drwxr-xr-x. 3 postgres postgres  76 May 21 00:32 base_000000010000000000000013_D_000000010000000000000011
-rw-r--r--. 1 postgres postgres 530 May 21 00:32 base_000000010000000000000013_D_000000010000000000000011_backup_stop_sentinel.json
drwxr-xr-x. 3 postgres postgres  76 May 21 00:39 base_000000010000000000000016_D_000000010000000000000013
-rw-r--r--. 1 postgres postgres 528 May 21 00:39 base_000000010000000000000016_D_000000010000000000000013_backup_stop_sentinel.json



#### 1.12) restore
rm -rf /var/lib/pgsql/15/data
wal-g backup-fetch /var/lib/pgsql/15/data LATEST


#### 2)

#### Задание повышенной сложности* под нагрузкой* бэкап снимаем с реплики**

<pre>

</pre>

