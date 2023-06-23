#### 0)


думаем как найти общий вход
https://www.percona.com/blog/postgresql-application-connection-failover-using-haproxy-with-xinetd/
https://www.percona.com/blog/postgresql-application-connection-failover-using-haproxy-with-xinetd/
https://github.com/hapostgres/pg_auto_failover/issues/826
https://github.com/neuroforgede/pg_auto_failover_ansible/wiki/HAProxy

===========================================================================================================================
su - postgres
su - postgres
su - postgres

Выполните следующую команду, чтобы запустить сервер монитора пользователем postgres.
/usr/pgsql-15/bin/postgres -D /var/lib/pgsql/15/data -p 5432 -h * &

стартуем службу pg_autoctl на каждом сервере и мониторе
/usr/pgsql-15/bin/pg_autoctl run &

статус
/usr/pgsql-15/bin/pg_autoctl show state
===========================================================================================================================


#### 0) Для ДЗ




 
Задание повышенной сложности*
Создать два кластера GKE в разных регионах
Установить на первом Patroni HA кластер
Установить на втором Patroni Standby кластер
Настроить TCP LB между регионами
Сделать в каждом регионе по клиентской ВМ
Проверить как ходит трафик с клиентской ВМ
Описать что и как делали и с какими проблемами столкнулись
</pre>




#### Выбрать один из вариантов и развернуть кластер. Описать что и как делали и с какими проблемами столкнулись
#### Вариант 1 How to Deploy PostgreSQL for High Availability
#### Вариант 2 Introducing pg_auto_failover: Open source extension for automated . failover and high-availability in PostgreSQL
#### Для гурманов Настройка Active/Passive PostgreSQL Cluster с использованием Pacemaker, Corosync, и DRBD (CentOS 5,5)


#### Настраиваю high-availability in PostgreSQL на pg_auto_failover

1
<pre> 
sudo yum -y install epel-release
yum install -y htop mc vim wget telnet

sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm && yum -y repolist && yum -y install postgresql15-contrib

или

sudo yum -y repolist
sudo yum install postgresql15-contrib
</pre> 


<pre> 
настраиваю по схеме:
Приложение > HAproxy keepalive cluster > проверка мастера xinetd > Pgbouncer > PostgreSQL (pg_autoctl)

что настраиваем:
pg1.ru-central1.internal       —> основной узел  PostgreSQL(pg_auto_failover) 1 + pgbouncer + служба xinetd проверка мастера xinetd
pg2.ru-central1.internal       —> вторичный узел PostgreSQL(pg_auto_failover) 2 + pgbouncer + служба xinetd проверка мастера xinetd
pgmonitor.ru-central1.internal —> узел монитора pg_auto_failover который действует как свидетель и оркестратор.
haproxi1.ru-central1.internal  —> узел 1 haproxi c сервисом keepalive
haproxi2.ru-central1.internal  —> узел 2 haproxi c сервисом keepalive
</pre> 

настроим монитор (только на pgmonitor):
Нам нужно сначала настроить pgmonitor, так как он будет периодически контролировать узлы БД и следить за их здоровьем. 
Для полной настройки мы будем использовать команды pg_autoctl вместе с подкомандами для соответствующих операций.

Создаем свои каталоги данных и tmp (если они не существуют) и измените владельца на пользователя postgres.
mkdir -p /tmp/pg_autoctl
chown -R postgres:postgres  /tmp/pg_autoctl
mkdir -p /var/lib/pgsql/15/data
chown -R postgres:postgres  /var/lib/pgsql/15/data

3аметка: 
убедитесь, что в вашей системе есть пользовательский postgres.
Если нет, создаем его перед выполнением вышеуказанных шагов.
выполняем приведенную ниже команду, чтобы настроить монитор.

yum install pg_auto_failover_15
yum install pg_auto_failover_15
yum install pg_auto_failover_15


выполняем приведенную ниже команду, чтобы настроить монитор.
/usr/pgsql-15/bin/pg_autoctl create monitor --auth trust --ssl-self-signed --pgdata /var/lib/pgsql/15/data/ --pgctl /usr/pgsql-15/bin/pg_ctl
/usr/pgsql-15/bin/pg_autoctl create monitor --auth trust --ssl-self-signed --pgdata /var/lib/pgsql/15/data/ --pgctl /usr/pgsql-15/bin/pg_ctl
/usr/pgsql-15/bin/pg_autoctl create monitor --auth trust --ssl-self-signed --pgdata /var/lib/pgsql/15/data/ --pgctl /usr/pgsql-15/bin/pg_ctl

Где:
pgdata —> Абсолютный путь к каталогу данных
pgctl  —> Абсолютный путь к бинарному файлу pg_ctl

под postgres
su - postgres
-bash-4.2$ /usr/pgsql-15/bin/pg_autoctl create monitor --auth trust --ssl-self-signed --pgdata /var/lib/pgsql/15/data/ --pgctl /usr/pgsql-15/bin/pg_ctl
-bash-4.2$ /usr/pgsql-15/bin/pg_autoctl create monitor --auth trust --ssl-self-signed --pgdata /var/lib/pgsql/15/data/ --pgctl /usr/pgsql-15/bin/pg_ctl
-bash-4.2$ /usr/pgsql-15/bin/pg_autoctl create monitor --auth trust --ssl-self-signed --pgdata /var/lib/pgsql/15/data/ --pgctl /usr/pgsql-15/bin/pg_ctl

08:12:57 9478 INFO  Using default --ssl-mode "require"
08:12:57 9478 INFO  Using --ssl-self-signed: pg_autoctl will create self-signed certificates, allowing for encrypted network traffic
08:12:57 9478 WARN  Self-signed certificates provide protection against eavesdropping; this setup does NOT protect against Man-In-The-Middle attacks nor Impersonation attacks.
08:12:57 9478 WARN  See https://www.postgresql.org/docs/current/libpq-ssl.html for details
08:12:57 9478 INFO  Initialising a PostgreSQL cluster at "/var/lib/pgsql/15/data"
08:12:57 9478 INFO  /usr/pgsql-15/bin/pg_ctl initdb -s -D /var/lib/pgsql/15/data --option '--auth=trust'
08:12:59 9478 INFO   /bin/openssl req -new -x509 -days 365 -nodes -text -out /var/lib/pgsql/15/data/server.crt -keyout /var/lib/pgsql/15/data/server.key -subj "/CN=pgmonitor.ru-central1.internal"
08:12:59 9478 INFO  Started pg_autoctl postgres service with pid 9500
08:12:59 9500 INFO   /usr/pgsql-15/bin/pg_autoctl do service postgres --pgdata /var/lib/pgsql/15/data/ -v
08:12:59 9478 INFO  Started pg_autoctl monitor-init service with pid 9501
08:12:59 9505 INFO   /usr/pgsql-15/bin/postgres -D /var/lib/pgsql/15/data -p 5432 -h *
08:12:59 9501 WARN  Failed to connect to "postgres://@:5432/postgres?", retrying until the server is ready
08:12:59 9501 WARN  The server at "postgres://@:5432/postgres?" is running but is in a state that disallows connections (startup, shutdown, or crash recovery).
08:12:59 9501 INFO  Successfully connected to "postgres://@:5432/postgres?" after 5 attempts in 107 ms.
08:12:59 9501 WARN  NOTICE:  installing required extension "btree_gist"
08:12:59 9501 INFO  Granting connection privileges on 10.129.0.0/24
08:12:59 9501 WARN  Skipping HBA edits (per --skip-pg-hba) for rule: hostssl "pg_auto_failover" "autoctl_node" 10.129.0.0/24 trust
08:12:59 9501 INFO  Your pg_auto_failover monitor instance is now ready on port 5432.
08:12:59 9501 INFO  Monitor has been successfully initialized.
08:12:59 9500 INFO  Postgres is now serving PGDATA "/var/lib/pgsql/15/data" on port 5432 with pid 9505
08:12:59 9478 WARN  pg_autoctl service monitor-init exited with exit status 0
08:12:59 9500 INFO  Postgres controller service received signal SIGTERM, terminating
08:12:59 9500 INFO  Stopping pg_autoctl postgres service
08:12:59 9500 INFO  /usr/pgsql-15/bin/pg_ctl --pgdata /var/lib/pgsql/15/data --wait stop --mode fast
08:13:00 9478 INFO  Waiting for subprocesses to terminate.
08:13:00 9478 INFO  Stop pg_autoctl

Он также устанавливает расширение pgautofailover и предоставляет доступ новому пользователю autoctl_node в локальной базе данных PostgreSQL.
Примечание. Сервер мониторинга также запускает службу PostgreSQL для сохранения 
изменений состояния серверов баз данных (основных или вторичных) только для своих собственных целей. 

Выполните следующую команду, чтобы запустить сервер монитора пользователем postgres.
su - postgres
/usr/pgsql-15/bin/postgres -D /var/lib/pgsql/15/data -p 5432 -h * &
/usr/pgsql-15/bin/postgres -D /var/lib/pgsql/15/data -p 5432 -h * &
/usr/pgsql-15/bin/postgres -D /var/lib/pgsql/15/data -p 5432 -h * &

проверяем войдя в службу PostgreSQL (пользователем postgres)
psql -d pg_auto_failover

Где:
-d —> pg_auto_failover — это база данных, созданная при настройке монитора.


стартуем службу pg_autoctl в качестве монитора (сервера монитора) под пользователем postgres.
/usr/pgsql-15/bin/pg_autoctl run &
/usr/pgsql-15/bin/pg_autoctl run &
/usr/pgsql-15/bin/pg_autoctl run &

Он начинает прослушивать порт 5432 на сервере мониторинга для дальнейшей настройки и мониторинга.
[1] 10378
-bash-4.2$ 08:24:27 10378 INFO  Started pg_autoctl postgres service with pid 10380
08:24:27 10380 INFO   /usr/pgsql-15/bin/pg_autoctl do service postgres --pgdata /var/lib/pgsql/15/data -v
08:24:27 10378 INFO  Started pg_autoctl listener service with pid 10381
08:24:27 10381 INFO   /usr/pgsql-15/bin/pg_autoctl do service listener --pgdata /var/lib/pgsql/15/data -v
08:24:27 10381 INFO  Managing the monitor at postgres://autoctl_node@pgmonitor.ru-central1.internal:5432/pg_auto_failover?sslmode=require
08:24:27 10381 INFO  Reloaded the new configuration from "/var/lib/pgsql/.config/pg_autoctl/var/lib/pgsql/15/data/pg_autoctl.cfg"
08:24:27 10381 INFO  Reloading Postgres configuration and HBA rules
08:24:27 10380 WARN  Postgres is already running with pid 9833, which is not a sub-process of pg_autoctl, restarting Postgres
08:24:27 10380 INFO  Stopping pg_autoctl postgres service
08:24:27 10380 INFO  /usr/pgsql-15/bin/pg_ctl --pgdata /var/lib/pgsql/15/data --wait stop --mode fast
08:24:27 10389 INFO   /usr/pgsql-15/bin/postgres -D /var/lib/pgsql/15/data -p 5432 -h *
08:24:27 10380 INFO  Postgres is now serving PGDATA "/var/lib/pgsql/15/data" on port 5432 with pid 10389
08:24:27 10380 WARN  PostgreSQL had to be stopped and restarted, it is now running as a subprocess of pg_autoctl, with pid 10389
08:24:28 10381 INFO  The version of extension "pgautofailover" is "2.0" on the monitor
08:24:28 10381 INFO  Contacting the monitor to LISTEN to its events.


--------------------------------------------------------------------------------------------------------------------
добавим разрешающее подключение для серверов кластера на сервере мониторе pgmonitor.ru-central1.internal в pg_hba.conf
host    pg_auto_failover  autoctl_node             ip_pg1.ru-central1.internal            md5/SCRAM-SHA-256
host    pg_auto_failover  autoctl_node             ip_pg2.ru-central1.internal            md5/SCRAM-SHA-256
--------------------------------------------------------------------------------------------------------------------


После успешной настройки сервера монитора переходим к настройке сервера БД.
Настройка серверов БД (только на серверах БД):
Следующая команда будет выполнена на pg1 для настройки первичной базы данных.
sudo -u postgres /usr/pgsql-15/bin/pg_autoctl create postgres --pgdata /var/lib/pgsql/15/data/ --auth trust --ssl-self-signed --username vorori --dbname mydb --hostname pg1.ru-central1.internal --pgctl /usr/pgsql-15/bin/pg_ctl --monitor 'postgres://autoctl_node@pgmonitor.ru-central1.internal:5432/pg_auto_failover?sslmode=require'
sudo -u postgres /usr/pgsql-15/bin/pg_autoctl create postgres --pgdata /var/lib/pgsql/15/data/ --auth trust --ssl-self-signed --username vorori --dbname mydb --hostname pg1.ru-central1.internal --pgctl /usr/pgsql-15/bin/pg_ctl --monitor 'postgres://autoctl_node@pgmonitor.ru-central1.internal:5432/pg_auto_failover?sslmode=require'
sudo -u postgres /usr/pgsql-15/bin/pg_autoctl create postgres --pgdata /var/lib/pgsql/15/data/ --auth trust --ssl-self-signed --username vorori --dbname mydb --hostname pg1.ru-central1.internal --pgctl /usr/pgsql-15/bin/pg_ctl --monitor 'postgres://autoctl_node@pgmonitor.ru-central1.internal:5432/pg_auto_failover?sslmode=require'

Где:
pgdata           —> Абсолютный путь к каталогу данных
имя пользователя —> Пользователь, который будет создан в базе данных
dbname           —> База данных, которая будет создана в базе данных 
имя хоста        —> Имя хоста первичной базы данных (Текущий сервер)
pg_ctl           —> Абсолютный путь к двоичному файлу pg_ctl

autoctl_node                        —> Пользователь по умолчанию для подключения к серверу монитора. Он будет создан на сервере монитора во время настройки, как упоминалось ранее.
pgmonitor.ru-central1.internal:5432 —> Имя хоста или IP-адрес сервера монитора вместе с портом.
pg_auto_failover                    —> База данных, созданная на сервере мониторинга во время настройки.


08:50:06 18615 INFO  Using default --ssl-mode "require"
08:50:06 18615 INFO  Using --ssl-self-signed: pg_autoctl will create self-signed certificates, allowing for encrypted network traffic
08:50:06 18615 WARN  Self-signed certificates provide protection against eavesdropping; this setup does NOT protect against Man-In-The-Middle attacks nor Impersonation attacks.
08:50:06 18615 WARN  See https://www.postgresql.org/docs/current/libpq-ssl.html for details
08:50:06 18615 INFO  Started pg_autoctl postgres service with pid 18617
08:50:06 18617 INFO   /usr/pgsql-15/bin/pg_autoctl do service postgres --pgdata /var/lib/pgsql/15/data/ -v
08:50:06 18615 INFO  Started pg_autoctl node-init service with pid 18618
08:50:06 18618 WARN  Failed to connect to "postgres://autoctl_node@pgmonitor.ru-central1.internal:5432/pg_auto_failover?sslmode=require", retrying until the server is ready
08:59:47 18618 INFO  Successfully connected to "postgres://autoctl_node@pgmonitor.ru-central1.internal:5432/pg_auto_failover?sslmode=require" after 335 attempts in 580576 ms.
08:59:47 18618 INFO  Registered node 1 "node_1" (pg1.ru-central1.internal:5432) in formation "default", group 0, state "single"
08:59:47 18618 INFO  Writing keeper state file at "/var/lib/pgsql/.local/share/pg_autoctl/var/lib/pgsql/15/data/pg_autoctl.state"
08:59:47 18618 INFO  Writing keeper init state file at "/var/lib/pgsql/.local/share/pg_autoctl/var/lib/pgsql/15/data/pg_autoctl.init"
08:59:47 18618 INFO  Successfully registered as "single" to the monitor.
08:59:47 18618 INFO  FSM transition from "init" to "single": Start as a single node
08:59:47 18618 INFO  Initialising postgres as a primary
08:59:47 18618 INFO  Initialising a PostgreSQL cluster at "/var/lib/pgsql/15/data"
08:59:47 18618 INFO  /usr/pgsql-15/bin/pg_ctl initdb -s -D /var/lib/pgsql/15/data --option '--auth=trust'
08:59:50 18618 WARN  could not change directory to "/home/vorori": Permission denied
08:59:50 18618 INFO   /bin/openssl req -new -x509 -days 365 -nodes -text -out /var/lib/pgsql/15/data/server.crt -keyout /var/lib/pgsql/15/data/server.key -subj "/CN=pg1.ru-central1.internal"
08:59:50 18740 INFO   /usr/pgsql-15/bin/postgres -D /var/lib/pgsql/15/data -p 5432 -h *
08:59:50 18618 INFO  CREATE DATABASE mydb;
08:59:50 18618 INFO  CREATE EXTENSION pg_stat_statements;
08:59:50 18617 INFO  Postgres is now serving PGDATA "/var/lib/pgsql/15/data" on port 5432 with pid 18740
08:59:50 18618 INFO  Disabling synchronous replication
08:59:50 18618 INFO  Reloading Postgres configuration and HBA rules
08:59:50 18618 INFO   /bin/openssl req -new -x509 -days 365 -nodes -text -out /var/lib/pgsql/15/data/server.crt -keyout /var/lib/pgsql/15/data/server.key -subj "/CN=pg1.ru-central1.internal"
08:59:50 18618 INFO  Contents of "/var/lib/pgsql/15/data/postgresql-auto-failover.conf" have changed, overwriting
08:59:50 18618 INFO  Reloading Postgres configuration and HBA rules
08:59:50 18618 INFO  Transition complete: current state is now "single"
08:59:51 18618 INFO  keeper has been successfully initialized.
08:59:51 18615 WARN  pg_autoctl service node-init exited with exit status 0
08:59:51 18617 INFO  Postgres controller service received signal SIGTERM, terminating
08:59:51 18617 INFO  Stopping pg_autoctl postgres service
08:59:51 18617 INFO  /usr/pgsql-15/bin/pg_ctl --pgdata /var/lib/pgsql/15/data --wait stop --mode fast
08:59:51 18615 INFO  Waiting for subprocesses to terminate.
08:59:51 18615 INFO  Stop pg_autoctl


стартуем службу pg_autoctl в качестве хранителя (сервера БД) под пользователем postgres.
/usr/pgsql-15/bin/pg_autoctl run &
/usr/pgsql-15/bin/pg_autoctl run &
/usr/pgsql-15/bin/pg_autoctl run &


su - postgres
-bash-4.2$ /usr/pgsql-15/bin/pg_autoctl run &
[1] 18879
-bash-4.2$ 09:04:15 18879 INFO  Started pg_autoctl postgres service with pid 18882
09:04:15 18882 INFO   /usr/pgsql-15/bin/pg_autoctl do service postgres --pgdata /var/lib/pgsql/15/data -v
09:04:15 18879 INFO  Started pg_autoctl node-active service with pid 18883
09:04:15 18883 INFO   /usr/pgsql-15/bin/pg_autoctl do service node-active --pgdata /var/lib/pgsql/15/data -v
09:04:15 18883 INFO  Reloaded the new configuration from "/var/lib/pgsql/.config/pg_autoctl/var/lib/pgsql/15/data/pg_autoctl.cfg"
09:04:15 18883 INFO  pg_autoctl service is running, current state is "single"
09:04:15 18891 INFO   /usr/pgsql-15/bin/postgres -D /var/lib/pgsql/15/data -p 5432 -h *
09:04:15 18882 INFO  Postgres is now serving PGDATA "/var/lib/pgsql/15/data" on port 5432 with pid 18891
09:04:15 18883 WARN  PostgreSQL was not running, restarted with pid 18891
09:04:16 18883 INFO  New state for this node (node 1, "node_1") (pg1.ru-central1.internal:5432): single ➜ single


Проверка текущего состояния
Наблюдаем статус сервера pg1.ru-central1.internal:5432 как единственный, поскольку в настоящее время доступен только один сервер.
-bash-4.2$ /usr/pgsql-15/bin/pg_autoctl show state
  Name |  Node |                     Host:Port |       TLI: LSN |   Connection |      Reported State |      Assigned State
-------+-------+-------------------------------+----------------+--------------+---------------------+--------------------
node_1 |     1 | pg1.ru-central1.internal:5432 |   1: 0/19D4448 |   read-write |              single |              single


делаем то же самое и на pg2.ru-central1.internal.
Здесь изменено только значение опции hostname. Использовался IP сервера pg2.ru-central1.internal.

sudo -u postgres /usr/pgsql-15/bin/pg_autoctl create postgres --pgdata /var/lib/pgsql/15/data/ --auth trust --ssl-self-signed --username vorori --dbname mydb --hostname pg2.ru-central1.internal --pgctl /usr/pgsql-15/bin/pg_ctl --monitor 'postgres://autoctl_node@pgmonitor.ru-central1.internal:5432/pg_auto_failover?sslmode=require'
sudo -u postgres /usr/pgsql-15/bin/pg_autoctl create postgres --pgdata /var/lib/pgsql/15/data/ --auth trust --ssl-self-signed --username vorori --dbname mydb --hostname pg1.ru-central1.internal --pgctl /usr/pgsql-15/bin/pg_ctl --monitor 'postgres://autoctl_node@pgmonitor.ru-central1.internal:5432/pg_auto_failover?sslmode=require'
sudo -u postgres /usr/pgsql-15/bin/pg_autoctl create postgres --pgdata /var/lib/pgsql/15/data/ --auth trust --ssl-self-signed --username vorori --dbname mydb --hostname pg1.ru-central1.internal --pgctl /usr/pgsql-15/bin/pg_ctl --monitor 'postgres://autoctl_node@pgmonitor.ru-central1.internal:5432/pg_auto_failover?sslmode=require'

09:16:28 16492 INFO  Using default --ssl-mode "require"
09:16:28 16492 INFO  Using --ssl-self-signed: pg_autoctl will create self-signed certificates, allowing for encrypted network traffic
09:16:28 16492 WARN  Self-signed certificates provide protection against eavesdropping; this setup does NOT protect against Man-In-The-Middle attacks nor Impersonation attacks.
09:16:28 16492 WARN  See https://www.postgresql.org/docs/current/libpq-ssl.html for details
09:16:28 16492 INFO  Started pg_autoctl postgres service with pid 16494
09:16:28 16494 INFO   /usr/pgsql-15/bin/pg_autoctl do service postgres --pgdata /var/lib/pgsql/15/data/ -v
09:16:28 16492 INFO  Started pg_autoctl node-init service with pid 16495
09:16:28 16495 INFO  Registered node 2 "node_2" (pg2.ru-central1.internal:5432) in formation "default", group 0, state "wait_standby"
09:16:28 16495 INFO  Writing keeper state file at "/var/lib/pgsql/.local/share/pg_autoctl/var/lib/pgsql/15/data/pg_autoctl.state"
09:16:28 16495 INFO  Writing keeper init state file at "/var/lib/pgsql/.local/share/pg_autoctl/var/lib/pgsql/15/data/pg_autoctl.init"
09:16:28 16495 INFO  Successfully registered as "wait_standby" to the monitor.
09:16:28 16495 INFO  FSM transition from "init" to "wait_standby": Start following a primary
09:16:28 16495 INFO  Transition complete: current state is now "wait_standby"
09:16:28 16495 INFO  New state for node 1 "node_1" (pg1.ru-central1.internal:5432): single ➜ wait_primary
09:16:29 16495 INFO  New state for node 1 "node_1" (pg1.ru-central1.internal:5432): wait_primary ➜ wait_primary
09:16:29 16495 INFO  FSM transition from "wait_standby" to "catchingup": The primary is now ready to accept a standby
09:16:29 16495 INFO  Initialising PostgreSQL as a hot standby
09:16:29 16495 INFO   /usr/pgsql-15/bin/pg_basebackup -w -d 'application_name=pgautofailover_standby_2 host=pg1.ru-central1.internal port=5432 user=pgautofailover_replicator sslmode=require' --pgdata /var/lib/pgsql/15/backup/node_2 -U pgautofailover_replicator --verbose --progress --max-rate 100M --wal-method=stream --slot pgautofailover_standby_2
09:16:29 16495 INFO  could not change directory to "/home/vorori": Permission denied
09:16:29 16495 INFO  pg_basebackup: initiating base backup, waiting for checkpoint to complete
09:16:29 16495 INFO  pg_basebackup: checkpoint completed
09:16:29 16495 INFO  pg_basebackup: write-ahead log start point: 0/2000060 on timeline 1
09:16:29 16495 INFO  pg_basebackup: starting background WAL receiver
09:16:29 16495 INFO  31168/31168 kB (100%), 0/1 tablespace (.../backup/node_2/global/pg_control)
09:16:29 16495 INFO  31168/31168 kB (100%), 1/1 tablespace
09:16:29 16495 INFO  pg_basebackup: write-ahead log end point: 0/2000138
09:16:29 16495 INFO  pg_basebackup: waiting for background process to finish streaming ...
09:16:29 16495 INFO  pg_basebackup: syncing data to disk ...
09:16:31 16495 INFO  pg_basebackup: renaming backup_manifest.tmp to backup_manifest
09:16:31 16495 INFO  pg_basebackup: base backup completed
09:16:31 16495 INFO  Creating the standby signal file at "/var/lib/pgsql/15/data/standby.signal", and replication setup at "/var/lib/pgsql/15/data/postgresql-auto-failover-standby.conf"
09:16:31 16495 INFO   /bin/openssl req -new -x509 -days 365 -nodes -text -out /var/lib/pgsql/15/data/server.crt -keyout /var/lib/pgsql/15/data/server.key -subj "/CN=pg2.ru-central1.internal"
09:16:31 16503 INFO   /usr/pgsql-15/bin/postgres -D /var/lib/pgsql/15/data -p 5432 -h *
09:16:32 16495 INFO  PostgreSQL started on port 5432
09:16:32 16495 INFO  Fetched current list of 1 other nodes from the monitor to update HBA rules, including 1 changes.
09:16:32 16495 INFO  Ensuring HBA rules for node 1 "node_1" (pg1.ru-central1.internal:5432)
09:16:32 16495 INFO  Adding HBA rule: hostssl replication "pgautofailover_replicator" pg1.ru-central1.internal trust
09:16:32 16495 INFO  Adding HBA rule: hostssl "mydb" "pgautofailover_replicator" pg1.ru-central1.internal trust
09:16:32 16495 INFO  Writing new HBA rules in "/var/lib/pgsql/15/data/pg_hba.conf"
09:16:32 16495 INFO  Reloading Postgres configuration and HBA rules
09:16:32 16494 INFO  Postgres is now serving PGDATA "/var/lib/pgsql/15/data" on port 5432 with pid 16503
09:16:32 16495 INFO  Transition complete: current state is now "catchingup"
09:16:32 16495 INFO  keeper has been successfully initialized.
09:16:32 16492 WARN  pg_autoctl service node-init exited with exit status 0
09:16:32 16494 INFO  Postgres controller service received signal SIGTERM, terminating
09:16:32 16494 INFO  Stopping pg_autoctl postgres service
09:16:32 16494 INFO  /usr/pgsql-15/bin/pg_ctl --pgdata /var/lib/pgsql/15/data --wait stop --mode fast
09:16:32 16492 INFO  Waiting for subprocesses to terminate.
09:16:32 16492 INFO  Stop pg_autoctl

Затем выполняем
su - postgres
/usr/pgsql-15/bin/pg_autoctl run &

bash-4.2$ 09:17:44 16545 INFO  Started pg_autoctl postgres service with pid 16548
09:17:44 16548 INFO   /usr/pgsql-15/bin/pg_autoctl do service postgres --pgdata /var/lib/pgsql/15/data -v
09:17:44 16545 INFO  Started pg_autoctl node-active service with pid 16549
09:17:44 16549 INFO   /usr/pgsql-15/bin/pg_autoctl do service node-active --pgdata /var/lib/pgsql/15/data -v
09:17:45 16549 INFO  Reloaded the new configuration from "/var/lib/pgsql/.config/pg_autoctl/var/lib/pgsql/15/data/pg_autoctl.cfg"
09:17:45 16549 INFO  pg_autoctl service is running, current state is "catchingup"
09:17:45 16549 WARN  Postgres is not running and we are in state catchingup
09:17:45 16549 WARN  Failed to update the keeper's state from the local PostgreSQL instance.
09:17:45 16549 INFO  Fetched current list of 1 other nodes from the monitor to update HBA rules, including 1 changes.
09:17:45 16549 INFO  Ensuring HBA rules for node 1 "node_1" (pg1.ru-central1.internal:5432)
09:17:45 16557 INFO   /usr/pgsql-15/bin/postgres -D /var/lib/pgsql/15/data -p 5432 -h *
09:17:45 16549 WARN  PostgreSQL was not running, restarted with pid 16557
09:17:45 16548 INFO  Postgres is now serving PGDATA "/var/lib/pgsql/15/data" on port 5432 with pid 16557
09:17:46 16549 INFO  Updated the keeper's state from the local PostgreSQL instance, which is running
09:17:46 16549 INFO  pg_autoctl managed to ensure current state "catchingup": PostgreSQL is running
09:17:47 16549 INFO  New state for this node (node 2, "node_2") (pg2.ru-central1.internal:5432): catchingup ➜ catchingup
09:17:47 16549 INFO  Monitor assigned new state "secondary"
09:17:47 16549 INFO  FSM transition from "catchingup" to "secondary": Convinced the monitor that I'm up and running, and eligible for promotion again
09:17:47 16549 INFO  Reached timeline 1, same as upstream node 1 "node_1" (pg1.ru-central1.internal:5432)
09:17:47 16549 INFO  Creating replication slot "pgautofailover_standby_1"
09:17:47 16549 INFO  Transition complete: current state is now "secondary"
09:17:47 16549 INFO  New state for this node (node 2, "node_2") (pg2.ru-central1.internal:5432): catchingup ➜ secondary
09:17:47 16549 INFO  New state for this node (node 2, "node_2") (pg2.ru-central1.internal:5432): secondary ➜ secondary
09:17:47 16549 INFO  New state for node 1 "node_1" (pg1.ru-central1.internal:5432): wait_primary ➜ primary
09:17:47 16549 INFO  New state for node 1 "node_1" (pg1.ru-central1.internal:5432): primary ➜ primary


При настройке pg2 pg_auto_failover также выполняет резервную настройку, выполняя pg_basebackup для начальной синхронизации.
теперь мы видим что в кластере появилась наша новая нода
-bash-4.2$ /usr/pgsql-15/bin/pg_autoctl show state
  Name |  Node |                     Host:Port |       TLI: LSN |   Connection |      Reported State |      Assigned State
-------+-------+-------------------------------+----------------+--------------+---------------------+--------------------
node_1 |     1 | pg1.ru-central1.internal:5432 |   1: 0/3000148 |   read-write |             primary |             primary
node_2 |     2 | pg2.ru-central1.internal:5432 |   1: 0/3000148 |    read-only |           secondary |           secondary


если надо удалить ноду из кластера: 
pg_autoctl drop node --destroy

если надо выполнить failover:
[postgres@pgmonitor] psql 
postgres=# \c pg_auto_failover
You are now connected to database "pg_auto_failover" as user "postgres".
pg_auto_failover=# select pgautofailover.perform_failover();
 perform_failover
------------------

(1 row)


видим что primary поменялся на pg2.ru-central1.internal:5432
-bash-4.2$ /usr/pgsql-15/bin/pg_autoctl show state
  Name |  Node |                     Host:Port |       TLI: LSN |   Connection |      Reported State |      Assigned State
-------+-------+-------------------------------+----------------+--------------+---------------------+--------------------
node_1 |     1 | pg1.ru-central1.internal:5432 |   2: 0/5000110 |    read-only |           secondary |           secondary
node_2 |     2 | pg2.ru-central1.internal:5432 |   2: 0/5000110 |   read-write |             primary |             primary


Между тем, HAProxy с xinetd даст нам роскошь увидеть, что является мастером, а что горячим резервом, 
чтобы соответствующим образом перенаправить соединения. 
Мы будем писать о встроенной проверке pgsql-check в следующих сообщениях блога и объяснять, как ее эффективно использовать.

Xinetd (Extended Internet Service Daemon) — это демон суперсервера. Он может прослушивать запросы на настраиваемых портах и 
​​отвечать на запросы, выполняя пользовательскую логику. На этот случай у нас есть собственные скрипты для проверки состояния базы данных. 
Используемый нами скрипт записывает HTTP-заголовок с кодом состояния. Другой код состояния представляет состояние экземпляра базы данных.
Код состояния 200, если экземпляр PostgreSQL является основным, 206, если PostgreSQL находится в режиме горячего резерва, и 503, 
если статус невозможно проверить.

На каждом сервере базы данных должна быть запущена служба xinetd на 
порту для проверки состояния работающих на нем экземпляров PostgreSQL. 
Обычно для этой цели используется порт: 23267, но мы можем использовать любой 
порт по нашему выбору. Этот сервис использует специально разработанный скрипт (скрипт оболочки) 
для определения 3 различных статусов экземпляров PostgreSQL.

_Первичная база данных
_Резервная база данных
_Невозможно подключиться к PostgreSQL — указание на то, что PostgreSQL не работает

Поскольку проверка состояния доступна через порт, предоставленный xinetd, 
HAProxy может отправить запрос на этот порт и понять состояние из ответа.

Установка и настройка:

создаю скрипт который будет говорить хапрокси кто у нас мастер а кто Standby
который может проверять состояние экземпляра PostgreSQL. Это довольно просто, скрипт оболочки вызывает утилиту psql
и выполняет функцию pg_is_in_recovery() postgres. 
По результату он может понять, является ли он ведущим или ведомым, или ему не удалось подключиться.


mkdir /opt/scripts/
vim /opt/scripts/pgsqlchk.sh
sudo chmod 755 /opt/scripts/pgsqlchk.sh

----------------------------------------------------------------------------------------------------------------------------------
#!/bin/bash
# This script checks if a postgres server is healthy running on localhost. It will return:
# "HTTP/1.x 200 OK\r" (if postgres is running smoothly)
# - OR -
# "HTTP/1.x 500 Internal Server Error\r" (else)
# The purpose of this script is make haproxy capable of monitoring postgres properly
# It is recommended that a low-privileged postgres  user is created to be used by this script.
# For eg. create  user healthchkusr login password 'hc321';
 
PGBIN=/usr/pgsql-15/bin
PGSQL_HOST="localhost"
PGSQL_PORT="5432"
PGSQL_DATABASE="postgres"
PGSQL_USERNAME="postgres"
export PGPASSWORD="123456789"
TMP_FILE="/tmp/pgsqlchk.out"
ERR_FILE="/tmp/pgsqlchk.err"
 
 
# We perform a simple query that should return a few results
 
VALUE=`/usr/bin/psql -t -h localhost -U postgres -p 5432 -c "select pg_is_in_recovery()" 2> /dev/null`
# Check the output. If it is not empty then everything is fine and we return something. Else, we just do not return anything.
 
 
if [ $VALUE == "t" ]
then
    /bin/echo -e "HTTP/1.1 206 OK\r\n"
    /bin/echo -e "Content-Type: Content-Type: text/plain\r\n"
    /bin/echo -e "\r\n"
    /bin/echo "Standby"
    /bin/echo -e "\r\n"
elif [ $VALUE == "f" ]
then
    /bin/echo -e "HTTP/1.1 200 OK\r\n"
    /bin/echo -e "Content-Type: Content-Type: text/plain\r\n"
    /bin/echo -e "\r\n"
    /bin/echo "Primary"
    /bin/echo -e "\r\n"
else
    /bin/echo -e "HTTP/1.1 503 Service Unavailable\r\n"
    /bin/echo -e "Content-Type: Content-Type: text/plain\r\n"
    /bin/echo -e "\r\n"
    /bin/echo "DB Down"
    /bin/echo -e "\r\n"
fi
----------------------------------------------------------------------------------------------------------------------------------

Вместо аутентификации на основе пароля можно использовать любые методы аутентификации без пароля.
Теперь мы можем установить xinetd на сервер. 
sudo yum install -y xinetd telnet
sudo yum install -y xinetd telnet
sudo yum install -y xinetd telnet

конфигурируем xinetd
sudo vim /etc/xinetd.d/checkpostgres


service pgsqlchk
{
        flags           = REUSE
        socket_type     = stream
        port            = 23267
        wait            = no
        user            = nobody
        server          = /opt/scripts/pgsqlchk.sh
        log_on_failure  += USERID
        disable         = no
        only_from       = 0.0.0.0/0
        per_source      = UNLIMITED
}

Каталог /etc/xinetd.d/ содержит файлы конфигурации для каждой службы, управляемой xinetd , и имена файлов 
соответствуют службе. Как и в случае с xinetd.conf ,
этот каталог читается только при запуске службы xinetd . Чтобы любые изменения вступили в силу,
администратор должен перезапустить службу xinetd .

Формат файлов в каталоге /etc/xinetd.d/ использует те же соглашения, что и /etc/xinetd.conf .
Основная причина, по которой конфигурация каждой службы хранится в отдельном файле,
состоит в том, чтобы сделать настройку проще и с меньшей вероятностью повлияет на другие службы.

добавим службу pgsqlchk в /etc/services.
sudo bash -c 'echo "pgsqlchk 23267/tcp # pgsqlchk" >> /etc/services'
sudo bash -c 'echo "pgsqlchk 23267/tcp # pgsqlchk" >> /etc/services'
sudo bash -c 'echo "pgsqlchk 23267/tcp # pgsqlchk" >> /etc/services'

стартуем
[root@pg2 scripts]# systemctl start xinetd
[root@pg1 scripts]# systemctl status xinetd
systemctl restart xinetd
systemctl stop xinetd

проверяем ччто служба слушает наш порт
[root@pg2 scripts]# sudo netstat -lntup | grep ":23267"
tcp6       0      0 :::23267                :::*                    LISTEN      10922/xinetd

[root@pg1 scripts]# sudo netstat -lntup | grep ":23267"
tcp6       0      0 :::23267                :::*                    LISTEN      10922/xinetd

устанавливаю настраиваю на сервера с postgre pgboucer
yum install pgbouncer
yum install pgbouncer
yum install pgbouncer

пред этими действиями в самом postgres мастере создаю пользователя integrator
alter user vorori with superuser; 
alter user vorori with password 'mypassword';                  --- vorori у меня уже есть просто накидываю ему права
create user integrator with password 'mypassword';
create database test with owner integrator;

------------------------------------------------------------------------------------------------------------------------------------------------
настраиваем конфигурацию на каждой ноде 
vim /etc/pgbouncer/pgbouncer.ini

[databases]
* = host=localhost port=5432

#описание:
----------------
* - означает дефолтную базу данных
host=localhost  соттветственно преадресуем на localhost (тут мы можем указать что угодно хоть проксировать запрос на другой срвер)
----------------


#обязательно выставляем:
listen_addr = *
listen_port = 6432
ignore_startup_parameters = extra_float_digits


#для нормальной работы подключения с бобра
ignore_startup_parameters = extra_float_digits


----------------
Смысл такой что для работы приложения надо оставлять работу
pool_mode = session 

авот для работы клиенских подключений надо выставлять 
pool_mode=transaction
----------------

описание:
----------------
Раздел [users]  [пользователи]
Этот раздел содержит строки ключ = значение, такие как

user1 = settings
где ключ будет восприниматься как имя пользователя, а значение — как список пар ключ=значение настроек конфигурации, специфичных для этого пользователя. Пример:

user1 = pool_mode=session
Здесь доступны только некоторые настройки.

pool_mode
Установите режим пула, который будет использоваться для всех подключений от этого ползователя. Если не задано, используется база данных или значение по умолчанию pool_mode.

max_user_connections
Настройте максимум для пользователя (т. е. все пулы с пользователем не будут иметь больше, чем это количество подключений к серверу).
----------------

max_client_conn = 300
auth_type = md5 # или SCRAM-SHA-256 
auth_file = /etc/pgbouncer/userlist.txt
------------------------------------------------------------------------------------------------------------------------------------------------


настраиваю userlist.txt
вставляем пароль пользователей (опять же скопировал уже в предыдушем шаге)
vim /etc/pgbouncer/userlist.txt
"vorori" "SCRAM-SHA-256$4096:wbQwgyduBNPbx76HdJw9uA==$pezbrqcrBDnEd/k3mITBhR4lSszWI+V8h0bFedDE5M4=:twBbKOlb80nh5Po+whwZ4RWFnMYFsNjMiL2FqG2dcXI="
"integrator" "SCRAM-SHA-256$4096:wbQwgyduBNPbx76HdJw9uA==$pezbrqcrBDnEd/k3mITBhR4lSszWI+V8h0bFedDE5M4=:twBbKOlb80nh5Po+whwZ4RWFnMYFsNjMiL2FqG2dcXI="


прегружаем службу балансирорвшика убеждаемся что все ок добавляем в автозагрузку
sudo systemctl enable --now pgbouncer
sudo service pgbouncer restart
sudo service pgbouncer status
sudo service pgbouncer stop
sudo service pgbouncer start


sudo netstat -lntup | grep ":6432"
sudo netstat -lntup | grep ":6432"
sudo netstat -lntup | grep ":6432"




haproxi1.ru-central1.internal  —> узел 1 haproxi c сервисом keepalive
haproxi2.ru-central1.internal  —> узел 2 haproxi c сервисом keepalive

Настройка HAProxy для использования xinetd
Нам нужно, чтобы HAProxy был установлен на сервере:


sudo yum install -y haproxy
sudo yum install -y haproxy
sudo yum install -y haproxy

конфигурациЯ HAProxy. 
sudo vim /etc/haproxy/haproxy.cfg


------------------------------------------------------------------------	
global
     log 127.0.0.1   local2
     log /dev/log    local0
     log /dev/log    local1 notice
     chroot /var/lib/haproxy
     stats socket /var/lib/haproxy/stats
     stats timeout 30s
     user haproxy
     group haproxy
     maxconn 4000
     daemon

defaults
    mode                    tcp
    log                     global
    option                  tcplog
    retries                 3
    timeout queue           1m
    timeout connect         10s
    timeout client          30m
    timeout server          30m
    timeout check           10s
    maxconn                 3000

listen stats
    mode http
    bind *:7000
    stats enable
    stats uri /

listen primary_postgres_write
    bind *:5000
    option httpchk OPTIONS /master
    http-check expect status 200
    default-server inter 3s fall 3 rise 2 on-marked-down shutdown-sessions
    server pg1 pg1.ru-central1.internal:6432 maxconn 100 check port 23267
    server pg2 pg2.ru-central1.internal:6432 maxconn 100 check port 23267

listen standby_postgres_read
    bind *:5001
    balance leastconn
    option httpchk OPTIONS /replica
    http-check expect status 206
    default-server inter 3s fall 3 rise 2 on-marked-down shutdown-sessions
    server pg1 pg1.ru-central1.internal:6432 maxconn 100 check port 23267
    server pg2 pg2.ru-central1.internal:6432 maxconn 100 check port 23267
------------------------------------------------------------------------

В соответствии с приведенной выше конфигурацией ключевыми моментами, на которые следует обратить внимание, являются
HAProxy настроен на использование режима TCP
Служба HAProxy начнет прослушивать порты 5000 и 5001.
Порт 5000 предназначен для соединений для чтения и записи, а порт 5001 — для соединений только для чтения.
Проверка статуса выполняется с помощью функции http-check на порту 23267.
Оба сервера pg1 и pg2 являются кандидатами для подключения как для чтения-записи, так и для подключения только для чтения.
На основе http-проверки и возвращенного статуса он определяет текущую роль


запуск службы HAProxy.
sudo systemctl start haproxy
sudo systemctl status haproxy
sudo systemctl stop haproxy


Проверка и тестирование
Согласно конфигурации HAProxy, мы должны иметь доступ к порту 5000 для подключения для чтения и записи.
sudo netstat -lntup | grep ":5001"
sudo netstat -lntup | grep ":5000"


Для подключения только для чтения и записи мы должны иметь доступ к порту 5001:
[root@haproxi1 vorori]# psql -h localhost -p 5000 -U vorori -d postgres
Password for user vorori:
psql (15.3)
Type "help" for help.

postgres=# select pg_is_in_recovery();
 pg_is_in_recovery
-------------------
 f
(1 row)


Для подключения только для чтения мы должны иметь доступ к порту 5001:
[root@haproxi1 vorori]# psql -h localhost -p 5001 -U vorori -d postgres
Password for user vorori:
psql (15.3)
Type "help" for help.

postgres=# select pg_is_in_recovery();
 pg_is_in_recovery
-------------------
 t
(1 row)



Настроить keepalived на haproxi1.ru-central1.internal и haproxi2.ru-central1.internal

Плавающий IP адрес (или виртуальный, далее VIP) используется для обеспечения отказоустойчивости в кластерах.
Кластер конфигурируется таким образом, что плавающий IP присвоен только одному узлу
в каждый момент времени. Если узел с VIP стал недоступен, то VIP присваивается standby узлу.
Тогда standby узел посылает оповещение (gratuitous ARP), чтобы оповестить остальных участников сети
о смене MAC адреса интерфейса с присвоенным VIP.


Установим keepalived на оба узла:
yum install keepalived
yum install psmisc-22.20-17.el7.x86_64  # для скрипта killall -0 haproxy



Далее, нам нужно следующий параметр Linux Kernel, чтобы включить поддержку
плавающих IP. На обоих узлах:
echo "net.ipv4.ip_nonlocal_bind = 1" >> /etc/sysctl.conf
sysctl -p


Теперь займемся конфигурацией keepalived, поместим в файл назначим виртуальный ip из моей внутренней сети 10.129.0.15/24 
vim /etc/keepalived/keepalived.conf


! Configuration File for keepalived
 
vrrp_script chk_haproxy {
  script "killall -0 haproxy" # check the haproxy process
  interval 2                  # every 2 seconds
  weight 2                    # add 2 points if OK
}
 
vrrp_instance VI_1 {
  interface eth0              # interface to monitor тут надо указать имя сушествующего сетевого интерфейса!!!!
  state MASTER                # MASTER on haproxi1 , BACKUP on haproxy2
  virtual_router_id 51
  priority 101                # 101 on haproxi1 , 100 haproxy2
 
  virtual_ipaddress {
     10.129.0.15/24          # virtual ip address
  }
 
  track_script {
    chk_haproxy
  }
}


14)
проверим появился ли виртуальный ip адрес
systemctl enable --now keepalived.service
systemctl start keepalived.service
systemctl restart keepalived.service
systemctl status keepalived.service
systemctl stop keepalived.service

ip a
ip a
ip a

[root@haproxi1 vorori]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 100000
    link/ether d0:0d:10:f6:d6:6b brd ff:ff:ff:ff:ff:ff
    inet 10.129.0.10/24 brd 10.129.0.255 scope global noprefixroute eth0
       valid_lft forever preferred_lft forever
    inet 10.129.0.15/24 scope global secondary eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::d20d:10ff:fef6:d66b/64 scope link
       valid_lft forever preferred_lft forever

15)
подключаемся на вторую ноду haproxi2.ru-central1.internal 
vim /etc/keepalived/keepalived.conf


! Configuration File for keepalived
 
vrrp_script chk_haproxy {
  script "killall -0 haproxy" # check the haproxy process
  interval 2                  # every 2 seconds
  weight 2                    # add 2 points if OK
}
 
vrrp_instance VI_1 {
  interface eth0              # interface to monitor тут надо указать имя сушествующего сетевого интерфейса!!!!
  state MASTER                # MASTER on haproxi1 , BACKUP on haproxy2
  virtual_router_id 51
  priority 100                # 101 on haproxi1 , 100 haproxy2
 
  virtual_ipaddress {
     10.129.0.15/24          # virtual ip address
  }
 
  track_script {
    chk_haproxy
  }
}


16)
Описание:
killall -0 haproxy проверяет запущен ли процесс HAProxy.
Узел haproxi1.ru-central1.internal 101, поэтому он всегда будет "забирать" VIP себе, если жив.


17)
Проверка
Проверим, что VIP присвоился интерфейсу eth0. Выполним на на обоих серверах

ip addr | grep "inet" | grep "eth0"
    inet 10.129.0.10/24 brd 10.129.0.255 scope global noprefixroute eth0
    inet 10.129.0.15/24 scope global secondary eth0


ip addr | grep "inet" | grep "eth0"
    inet 10.129.0.19/24 brd 10.129.0.255 scope global noprefixroute eth0
    inet 10.129.0.15/24 scope global secondary eth0
	
	
все работает ура!


Заключение:
Это очень общий способ настройки HAProxy с кластером PostgreSQL, но он не ограничивается какой-либо конкретной топологией кластера. 
Проверка работоспособности выполняется с помощью пользовательского сценария оболочки, 
а результат проверки работоспособности доступен через службу xinetd. 
HAProxy использует эту информацию для обслуживания таблицы маршрутизации и перенаправления соединения на соответствующий узел в кластере.