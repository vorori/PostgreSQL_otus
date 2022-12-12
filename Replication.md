# 1

На 1 ВМ создаем таблицы test для записи, test2 для запросов на чтение. 
Создаем публикацию таблицы test и подписываемся на публикацию таблицы test2 с ВМ №2. 
На 2 ВМ создаем таблицы test2 для записи, test для запросов на чтение. 
Создаем публикацию таблицы test2 и подписываемся на публикацию таблицы test1 с ВМ №1. 
3 ВМ использовать как реплику для чтения и бэкапов (подписаться на таблицы из ВМ №1 и №2 ). Небольшое описание, того, что получилось.


<pre>
создал 4 vm
otus_vm1_centos7 ip 192.168.1.10
otus_vm2_centos7 ip 192.168.1.11
otus_vm3_centos7 ip 192.168.1.12
otus_vm4_centos7 ip 192.168.1.13

sudo /usr/pgsql-14/bin/postgresql-14-setup initdb
sudo systemctl enable --now postgresql-14
sudo systemctl status postgresql-14
sudo systemctl restart postgresql-14



#otus_vm1_centos7 ip 192.168.1.10
firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.1.11" port protocol="tcp" port="5432" accept"
firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.1.12" port protocol="tcp" port="5432" accept"
firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.1.13" port protocol="tcp" port="5432" accept"


#otus_vm2_centos7 ip 192.168.1.11
firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.1.10" port protocol="tcp" port="5432" accept"
firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.1.12" port protocol="tcp" port="5432" accept"
firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.1.13" port protocol="tcp" port="5432" accept"

#otus_vm3_centos7 ip 192.168.1.12
firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.1.10" port protocol="tcp" port="5432" accept"
firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.1.11" port protocol="tcp" port="5432" accept"
firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.1.13" port protocol="tcp" port="5432" accept"

#otus_vm4_centos7 ip 192.168.1.13
firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.1.10" port protocol="tcp" port="5432" accept"
firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.1.11" port protocol="tcp" port="5432" accept"
firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.1.12" port protocol="tcp" port="5432" accept"

firewall-cmd --reload		
firewall-cmd --list-all		
</pre>


<pre>
#на всех виртуалках применяю настройки
sudo -u postgres psql -c "ALTER SYSTEM SET listen_addresses = '*';"
sudo -u postgres psql -c "ALTER SYSTEM SET wal_level = 'logical';"
echo "host all all 192.168.1.0/24 md5" | sudo tee -a /var/lib/pgsql/14/data/pg_hba.conf
echo "host replication all 192.168.1.0/24 md5" | sudo tee -a /var/lib/pgsql/14/data/pg_hba.conf
sudo -u postgres psql -c "create user tehuser with password '123456789';"
sudo -u postgres psql -c "alter user tehuser with REPLICATION;"
sudo -u postgres psql mydb -c "GRANT SELECT ON ALL TABLES IN SCHEMA public TO tehuser;"
sudo systemctl restart postgresql-14
sudo systemctl status postgresql-14
</pre>

<pre>
На ВМ otus_vm1_centos7 создаем таблицы test для записи, test2 для запросов на чтение.
(otus_vm1_centos7 ip 192.168.1.10; otus_vm2_centos7 ip 192.168.1.11 ;otus_vm3_centos7 ip 192.168.1.12)

sudo -u postgres psql -c "CREATE DATABASE mydb"
sudo -u postgres psql mydb -c "CREATE TABLE test1 (m text)"
sudo -u postgres psql mydb -c "CREATE TABLE test2 (m text)"

sudo -u postgres psql mydb -c "select * from test1"
sudo -u postgres psql mydb -c "select * from test2"
</pre>

<pre>
Создаем публикацию таблицы test и подписываемся на публикацию таблицы test2 с ВМ №2. 

#на VM (otus_vm1_centos7 ip 192.168.1.10)
sudo -u postgres psql mydb -c "CREATE PUBLICATION test1_pub FOR TABLE test1"

#на VM (otus_vm2_centos7 ip 192.168.1.11)
sudo -u postgres psql mydb -c "CREATE SUBSCRIPTION test_sub CONNECTION 'host=192.168.1.10 port=5432 user=tehuser password=123456789 dbname=mydb' PUBLICATION test1_pub WITH (copy_data = true)"
</pre>


<pre>
На 2 ВМ создаем таблицы test2 для записи, test для запросов на чтение. 

#на VM (otus_vm2_centos7 ip 192.168.1.11)
sudo -u postgres psql mydb -c "CREATE PUBLICATION test2_pub FOR TABLE test2"

#на VM (otus_vm1_centos7 ip 192.168.1.10)
sudo -u postgres psql mydb -c "CREATE SUBSCRIPTION test2_sub CONNECTION 'host=192.168.1.11 port=5432 user=tehuser password=123456789 dbname=mydb' PUBLICATION test2_pub WITH (copy_data = true)"
</pre>

<pre>
#проверяем работу реплкации

#на VM (otus_vm1_centos7 ip 192.168.1.10)
sudo -u postgres psql mydb -c "INSERT INTO test1 VALUES ('Yo')"

#на VM (otus_vm2_centos7 ip 192.168.1.11)
sudo -u postgres psql mydb -c "select * from test1"


#на VM (otus_vm2_centos7 ip 192.168.1.11)
sudo -u postgres psql mydb -c "INSERT INTO test2 VALUES ('Ky')"

#на VM (otus_vm1_centos7 ip 192.168.1.10)
sudo -u postgres psql mydb -c "select * from test2"
</pre>


<pre>
3 ВМ использовать как реплику для чтения и бэкапов (подписаться на таблицы из ВМ №1 и №2 ). Небольшое описание, того, что получилось.

# на VM (otus_vm3_centos7 ip 192.168.1.12)
sudo -u postgres psql mydb -c "CREATE SUBSCRIPTION test_sub_for_otus_vm3 CONNECTION 'host=192.168.1.10 port=5432 user=tehuser password=123456789 dbname=mydb' PUBLICATION test1_pub WITH (copy_data = true)"
sudo -u postgres psql mydb -c "CREATE SUBSCRIPTION test2_sub_for_otus_vm3 CONNECTION 'host=192.168.1.11 port=5432 user=tehuser password=123456789 dbname=mydb' PUBLICATION test2_pub WITH (copy_data = true)"


Небольшое описание, того, что получилось.

У нас получилось репликация по схеме (обязательно чтобы это работало пользователь при помоши которого работает репликация должен иметь GRANT SELECT на реплицируемую таблицу):
vm1.test1 ---> vm2.test1 
vm2.test2 ---> vm1.test2

Реплика которая копирует все данные с vm1 и vm2
vm1.test1 ---> vm3.test1 
vm2.test2 ---> vm3.test2
</pre>

# 2
Задание со звездочкой*
реализовать горячее реплицирование для высокой доступности на 4ВМ. Источником должна выступать ВМ №3. Написать с какими проблемами столкнулись.

<pre>
####применяю на VM3 и VM4: (VM otus_vm3_centos7 ip 192.168.1.12 ; otus_vm4_centos7 ip 192.168.1.13)
sudo -u postgres psql -c "ALTER SYSTEM SET wal_level = 'replica';"
sudo systemctl status postgresql-14
sudo systemctl restart postgresql-14


###Теперь включаем на VM3 (он же будет мастером режим синхронной репликации)
sudo -u postgres psql -c "ALTER SYSTEM SET synchronous_commit TO 'on';"
####и обязательно добавляем этот параметр который указывает с какими репликами у нас будет выполнятся синхронная репликация (*) означает со всеми
sudo -u postgres psql -c "ALTER SYSTEM SET synchronous_standby_names TO '*';"


###создаю на VM3 мастере пользователя который будет отвечать за репликацию между VM3 и VM4
sudo -u postgres psql -c "CREATE ROLE user_replicator WITH REPLICATION PASSWORD 'klJlkdfsjImdsksdmsd98' LOGIN;"

####создаю слот на мастере VM3
sudo -u postgres psql -c "SELECT * FROM pg_create_physical_replication_slot('my_slot_replication');"

###проверяю слот
sudo -u postgres psql -c "SELECT * FROM pg_replication_slots;" \gx

SELECT * FROM pg_replication_slots \gx
postgres=# SELECT * FROM pg_replication_slots \gx
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




#на VM4 останавливаю кластер удаляю данные 
sudo systemctl stop postgresql-14
sudo -u postgres rm -rf /var/lib/pgsql/14/data/*
sudo -u postgres pg_basebackup -h 192.168.1.12 -D /var/lib/pgsql/14/data -U user_replicator -P -v -R -X stream -S my_slot_replication

проверяю что появилась дополнителная информация по слоту репликации
cat /var/lib/pgsql/14/data/postgresql.auto.conf

#запускаю кластер на VM4
sudo systemctl restart postgresql-14



#Переход на реплику
#Сейчас VM4сервер является репликой (находится в режиме восстановления):

sudo -u postgres psql -c "SELECT pg_is_in_recovery();"
sudo -u postgres psql mydb -c "select * from test1"
sudo -u postgres psql mydb -c "select * from test2"


#на главном сервере VM3 проверяем что репликация синхронная
Проверка физической репликации  sync_state: sync говорит о том, что репликация работает в синхронном режиме.
postgres=# SELECT * FROM pg_stat_replication \gx
-[ RECORD 1 ]----+------------------------------
pid              | 10183
usesysid         | 16398
usename          | user_replicator
application_name | walreceiver
client_addr      | 192.168.1.13
client_hostname  |
client_port      | 58830
backend_start    | 2022-12-12 17:14:03.385113+03
backend_xmin     |
state            | streaming
sent_lsn         | 0/7000488
write_lsn        | 0/7000488
flush_lsn        | 0/7000488
replay_lsn       | 0/7000488
write_lag        |
flush_lag        |
replay_lag       |
sync_priority    | 1
sync_state       | sync
reply_time       | 2022-12-12 17:14:23.433215+03




#Репликация работает вставляю на VM1 в таблицу 
sudo -u postgres psql mydb -c "INSERT INTO test1 VALUES ('zzzzzzzzzzzzzzzzz666666666666666666dsdsdsdsdsdsdsd')"
INSERT 0 1

и смотрю что на VM3 и соответственно VM4 этф вставка выполнилась все работает)
[root@localhost data]# sudo -u postgres psql mydb -c "select * from test1"
 Yo
 Yo00
 Yo0057
 666666666666666666
 666666666666666666
 zzzzzzzzzzzzzzzzz666666666666666666dsdsdsdsdsdsdsd
(6 rows)
</pre>

