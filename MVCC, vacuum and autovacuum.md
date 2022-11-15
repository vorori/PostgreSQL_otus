# 1
создать GCE инстанс типа e2-medium и диском 10GB

создал centos7 sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm

# 2
установить на него PostgreSQL 14 с дефолтными настройками

sudo yum install postgresql14-contrib 

sudo /usr/pgsql-14/bin/postgresql-14-setup initdb 

sudo systemctl enable --now postgresql-14 

systemctl status postgresql-14

# 3
применить параметры настройки PostgreSQL из прикрепленного к материалам занятия файла

в настройках есть параметр max_wal_size = 16GB который никак не вписывается в наши параметры сервера у нас та на сервере 10gb hdd


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


настройки которые я применил

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




# 4
выполнить pgbench -i postgres

# 5
запустить pgbench -c8 -P 60 -T 600 -U postgres postgres

# 6
дать отработать до конца

# 7
дальше настроить autovacuum максимально эффективно


# 8
построить график по получившимся значениям

# 9
так чтобы получить максимально ровное значение tps