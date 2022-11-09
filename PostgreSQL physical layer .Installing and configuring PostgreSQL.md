
# 1)
создайте виртуальную машину c Ubuntu 20.04 LTS (bionic) в GCE/ЯО

создал centos7
sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm

# 2)
поставьте на нее PostgreSQL 14 через sudo apt

sudo yum install postgresql14-contrib
sudo /usr/pgsql-14/bin/postgresql-14-setup initdb
sudo systemctl enable --now postgresql-14 
systemctl status postgresql-14 

# 3)
проверьте что кластер запущен через sudo -u postgres pg_lsclusters

systemctl status postgresql-14 
postgresql-14.service - PostgreSQL 14 database server
   Loaded: loaded (/usr/lib/systemd/system/postgresql-14.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2022-11-09 08:36:02 UTC; 14min ago

# 4)
зайдите из под пользователя postgres в psql и сделайте произвольную таблицу с произвольным содержимым


create table test(c1 text);
insert into test values('1');
\dt


# 5)
остановите postgres например через sudo -u postgres pg_ctlcluster 14 main stop

systemctl stop postgresql-14
systemctl status postgresql-14
● postgresql-14.service - PostgreSQL 14 database server
   Loaded: loaded (/usr/lib/systemd/system/postgresql-14.service; enabled; vendor preset: disabled)
   Active: inactive (dead) since Wed 2022-11-09 08:53:33 UTC; 6s ago

# 6)
создайте новый standard persistent диск GKE через Compute Engine -> Disks в том же регионе и зоне что GCE инстанс размером например 10GB

создал

# 7)
добавьте свеже-созданный диск к виртуальной машине - надо зайти в режим ее редактирования и дальше выбрать пункт attach existing disk

добавил

# 8)
проинициализируйте диск согласно инструкции и подмонтировать файловую систему, только не забывайте менять имя диска на актуальное, 
в вашем случае это скорее всего будет /dev/sdb - https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux


-----------------------

lsblk
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
vda    253:0    0  20G  0 disk
├─vda1 253:1    0   1M  0 part
└─vda2 253:2    0  20G  0 part /
vdb    253:16   0  10G  0 disk

sudo parted /dev/vdb mklabel gpt


sudo parted -a opt /dev/vdb mkpart primary ext4 0% 100%

lsblk
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
vda    253:0    0  20G  0 disk
├─vda1 253:1    0   1M  0 part
└─vda2 253:2    0  20G  0 part /
vdb    253:16   0  10G  0 disk
└─vdb1 253:17   0  10G  0 part


sudo lsblk -o NAME,FSTYPE,LABEL,UUID,MOUNTPOINT
NAME   FSTYPE LABEL         UUID                                 MOUNTPOINT
vda
├─vda1
└─vda2 xfs                  73c7a5bb-d81e-4815-8bef-f7b261426bab /
vdb
└─vdb1 ext4   datapartition a26f75e0-a2c3-4a90-b4b5-7b08353995f7


sudo lsblk -o NAME,FSTYPE,LABEL,UUID,MOUNTPOINT


sudo mkdir -p /mnt/data

sudo mount -o defaults /dev/vdb1 /mnt/data

df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        895M     0  895M   0% /dev
tmpfs           919M     0  919M   0% /dev/shm
tmpfs           919M  628K  919M   1% /run
tmpfs           919M     0  919M   0% /sys/fs/cgroup
/dev/vda2        20G  1.8G   19G   9% /
tmpfs           184M     0  184M   0% /run/user/1000
/dev/vdb1       9.8G   37M  9.2G   1% /mnt/data

смотрю параметры
sudo lsblk --f

добавляю информацию о новом диске
vim /etc/fstab
UUID=a26f75e0-a2c3-4a90-b4b5-7b08353995f7 /mnt/data               ext4    defaults        0 2

# 9)
перезагрузите инстанс и убедитесь, что диск остается примонтированным (если не так смотрим в сторону fstab)

reboot -h now
диск /dev/vdb1  остается после перезагрузки

df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        895M     0  895M   0% /dev
tmpfs           919M   28K  919M   1% /dev/shm
tmpfs           919M  604K  919M   1% /run
tmpfs           919M     0  919M   0% /sys/fs/cgroup
/dev/vda2        20G  1.8G   19G   9% /
/dev/vdb1       9.8G   37M  9.2G   1% /mnt/data
tmpfs           184M     0  184M   0% /run/user/1000


# 10)
сделайте пользователя postgres владельцем /mnt/data - chown -R postgres:postgres /mnt/data/

chown postgres:postgres /mnt/data

# 11)
перенесите содержимое /var/lib/postgres/14 в /mnt/data - mv /var/lib/postgresql/14 /mnt/data

systemctl stop postgresql-14
systemctl status postgresql-14
mv /var/lib/pgsql/14 /mnt/data   (Содержимое находится по пути /var/lib/pgsql/14)

# 12)
попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 14 main start

systemctl start postgresql-14

# 13)
напишите получилось или нет и почему

Кластер не стартует потому что мы преместили каталог с данными на другой диск в другую директорию

● postgresql-14.service - PostgreSQL 14 database server
   Loaded: loaded (/usr/lib/systemd/system/postgresql-14.service; enabled; vendor preset: disabled)
   Active: failed (Result: exit-code) since Wed 2022-11-09 12:08:37 UTC; 2min 24s ago
     Docs: https://www.postgresql.org/docs/14/static/
  Process: 897 ExecStart=/usr/pgsql-14/bin/postmaster -D ${PGDATA} (code=exited, status=0/SUCCESS)
  Process: 7693 ExecStartPre=/usr/pgsql-14/bin/postgresql-14-check-db-dir ${PGDATA} (code=exited, status=1/FAILURE)
 Main PID: 897 (code=exited, status=0/SUCCESS)



# 14)
задание: найти конфигурационный параметр в файлах раположенных в /etc/postgresql/14/main который надо поменять и поменяйте его

vim /usr/lib/systemd/system/postgresql-14.service
меняю значение пременной
Environment=PGDATA=/mnt/data/14/data

обязательно конфигурацию демона systemctl
systemctl daemon-reload  (презапускаю конфигурацию демона systemctl )

# 15)
напишите что и почему поменяли
менял пременную Environment в службе postgresql-14.service которая отвечает за местоположения каталога с данными postgres

# 16)
попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 14 main start

systemctl restart postgresql-14
systemctl status postgresql-14

● postgresql-14.service - PostgreSQL 14 database server
   Loaded: loaded (/usr/lib/systemd/system/postgresql-14.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2022-11-09 12:32:53 UTC; 11s ago
     Docs: https://www.postgresql.org/docs/14/static/
  Process: 8004 ExecStartPre=/usr/pgsql-14/bin/postgresql-14-check-db-dir ${PGDATA} (code=exited, status=0/SUCCESS)
 Main PID: 8010 (postmaster)
   CGroup: /system.slice/postgresql-14.service
           ├─8010 /usr/pgsql-14/bin/postmaster -D /mnt/data/14/data
           ├─8012 postgres: logger
           ├─8014 postgres: checkpointer
           ├─8015 postgres: background writer
           ├─8016 postgres: walwriter
           ├─8017 postgres: autovacuum launcher
           ├─8018 postgres: stats collector
           └─8019 postgres: logical replication launcher


# 17)
напишите получилось или нет и почему

все работает все получилось postgre сейчас работает на другом диске и служба обрашается на новый каталог с данными на новом диске

# 18)
зайдите через через psql и проверьте содержимое ранее созданной таблицы

sudo su - postgres
psql

-bash-4.2$ psql
psql (14.5)
Type "help" for help.

postgres= \dt
        List of relations
 Schema | Name | Type  |  Owner
--------+------+-------+----------
 public | test | table | postgres
(1 row)

postgres=# select * from test;
 c1
----
 1
(1 row)


# 19)
задание со звездочкой *: не удаляя существующий инстанс ВМ сделайте новый, поставьте на его PostgreSQL, 
удалите файлы с данными из /var/lib/postgres, перемонтируйте внешний диск который сделали ранее от первой 
виртуальной машины ко второй и запустите PostgreSQL на второй машине так чтобы он работал с данными на внешнем диске, 
расскажите как вы это сделали и что в итоге получилось.


Все получилось  кластер стартанул с примапленного диска с данными со старой виртуалки. такое задание очень понравилось !!! спасибо!)
сделал все тоже самое только диск не надо форматировать а монтировать с данными и тогда все ок)))

естественно пред тем как отмонтировать диск с первой виртуалки остановил службу postgres на первром сервере чтобы не повредить данные


создал директорию для монтирования нового диска
sudo mkdir -p /mnt/data
монтирую диск сразу (уже с данными от старого кластера с первой виртуалки)
sudo mount -o defaults /dev/vdb1 /mnt/data


[root@new vorori]# sudo mount -o defaults /dev/vdb1 /mnt/data
[root@new vorori]# df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        895M     0  895M   0% /dev
tmpfs           919M     0  919M   0% /dev/shm
tmpfs           919M  596K  919M   1% /run
tmpfs           919M     0  919M   0% /sys/fs/cgroup
/dev/vda2        20G  1.8G   19G   9% /
tmpfs           184M     0  184M   0% /run/user/1000
/dev/vdb1       9.8G   79M  9.2G   1% /mnt/data


редактирую службу заппуск чтобы указать директорию нового диска /mnt/data
vim /usr/lib/systemd/system/postgresql-14.service

меняю значение пременной
Environment=PGDATA=/mnt/data/14/data

обязательно конфигурацию демона systemctl
systemctl daemon-reload  (презапускаю конфигурацию демона systemctl )


systemctl start postgresql-14

[root@new data]# systemctl start postgresql-14
[root@new data]# systemctl status postgresql-14
● postgresql-14.service - PostgreSQL 14 database server
   Loaded: loaded (/usr/lib/systemd/system/postgresql-14.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2022-11-09 13:10:46 UTC; 9s ago
     Docs: https://www.postgresql.org/docs/14/static/
  Process: 8106 ExecStartPre=/usr/pgsql-14/bin/postgresql-14-check-db-dir ${PGDATA} (code=exited, status=0/SUCCESS)
 Main PID: 8112 (postmaster)
   CGroup: /system.slice/postgresql-14.service
           ├─8112 /usr/pgsql-14/bin/postmaster -D /mnt/data/14/data
           ├─8115 postgres: logger
           ├─8117 postgres: checkpointer
           ├─8118 postgres: background writer
           ├─8119 postgres: walwriter
           ├─8120 postgres: autovacuum launcher
           ├─8121 postgres: stats collector
           └─8122 postgres: logical replication launcher



root@new data] su - postgres
-bash-4.2$ psql
psql (14.5)
Type "help" for help.

postgres=# \dt
        List of relations
 Schema | Name | Type  |  Owner
--------+------+-------+----------
 public | test | table | postgres
(1 row)

postgres=# select * from test;
 c1
----
 1
(1 row)