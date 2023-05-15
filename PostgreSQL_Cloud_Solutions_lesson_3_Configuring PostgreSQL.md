#### 1)

#### создайте виртуальную машину c Ubuntu 20.04 LTS (bionic) в GCE типа e2-medium в default VPC в любом регионе и зоне, например us-central1-a или ЯО/VirtualBox


<pre>
создал vm centos 
</pre>

#### 2)

#### поставьте на нее PostgreSQL 15 через sudo apt

<pre>
sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
sudo yum -y repolist
sudo yum install postgresql15-contrib
</pre>

#### 3)

#### проверьте что кластер запущен через sudo -u postgres pg_lsclusters

<pre>
sudo /usr/pgsql-15/bin/postgresql-15-setup initdb
systemctl enable postgresql-15.service
systemctl status postgresql-15.service
systemctl restart postgresql-15.service
</pre>

#### 4)

####  зайдите из под пользователя postgres в psql и сделайте произвольную таблицу с произвольным содержимым
####  postgres=# create table test(c1 text);
####  postgres=# insert into test values('1');

<pre>
postgres=# \dt
        List of relations
 Schema | Name | Type  |  Owner
--------+------+-------+----------
 public | test | table | postgres
(1 row)
</pre>



#### 5)

####   остановите postgres например через sudo -u postgres pg_ctlcluster 15 main stop

<pre>
systemctl stop postgresql-15.service
systemctl status postgresql-15.service
● postgresql-15.service - PostgreSQL 15 database server
   Loaded: loaded (/usr/lib/systemd/system/postgresql-15.service; enabled; vendor preset: disabled)
   Active: inactive (dead) since Mon 2023-05-15 08:54:05 UTC; 7s ago
     Docs: https://www.postgresql.org/docs/15/static/
  Process: 8429 ExecStart=/usr/pgsql-15/bin/postmaster -D ${PGDATA} (code=exited, status=0/SUCCESS)
  Process: 8423 ExecStartPre=/usr/pgsql-15/bin/postgresql-15-check-db-dir ${PGDATA} (code=exited, status=0/SUCCESS)
 Main PID: 8429 (code=exited, status=0/SUCCESS)
</pre>



#### 6)

####  создайте новый standard persistent диск GKE через Compute Engine -> Disks в том же регионе и зоне что GCE 
####  инстанс размером например 10GB - или аналог в другом облаке/виртуализации

<pre>
диски --> создать диск --> 10 gb

создал
</pre>



#### 7)

####  добавьте свеже-созданный диск к виртуальной машине - надо зайти в режим ее редактирования и дальше выбрать пункт attach existing disk


<pre>
https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux

7.1
остановил vm --> добавил новый диск 10gb к VM --> запустил VM

7.2
использовать lsblk команду и найти диск нужного размера, который не имеет связанных разделов:
[root@postgre vorori]# lsblk
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
vda    253:0    0  20G  0 disk
├─vda1 253:1    0   1M  0 part
└─vda2 253:2    0  20G  0 part /
vdb    253:16   0  10G  0 disk


Посмотреть добавился ли диск очень легко воспользуемся командой: fdisk –l
fdisk -l

Здесь мы можем наблюдать оба наших диска, где:

\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\
Disk /dev/vda: 21.5 GB, 21474836480 bytes, 41943040 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disk label type: gpt
Disk identifier: 01791D85-D081-4F33-BADF-3DE986DA6219


#         Start          End    Size  Type            Name
 1         2048         4095      1M  BIOS boot
 2         4096     41943005     20G  Microsoft basic
\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\

//////////////////////////////////////////////////////////////////////////////////////
Disk /dev/vdb: 10.7 GB, 10737418240 bytes, 20971520 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 8192 bytes
I/O size (minimum/optimal): 8192 bytes / 8192 bytes
//////////////////////////////////////////////////////////////////////////////////////


Disk /dev/vda – наш изначальный диск, где установлена система, размечен swap и загрузочный раздел.
Disk /dev/vdb – Это наш новый диск, который мы и будем далее размечать.


7.3
-Далее на используемом диске создадим раздел, так же через утилиту fdisk.
-Для этого используем утилиту fdisk и укажем имя диска на котором будем создавать раздел т.е.

fdisk /dev/vdb

-Утилита fdisk встречает нас приветственным сообщением, далее нам необходимо указать ключ для создания нового -раздела. Укажем «n».

n

Далее нам надо выбрать тип нового раздела первичный(primary) или логический extended. Выбираем первичный.

p

Далее нам необходимо указать номер раздела укажем 1.

1

На этом шаге нам необходимо указать начало раздела, оставим по умолчанию и нажмем enter.
First sector (2048-20971519, default 2048):

enter


Далее утилита предлагает нам выбрать окончание нового раздела, так же оставляем по умолчанию и нажимаем Enter.

Enter


Будет создан новый раздел размером 10 гигабайт. Теперь нам необходимо сохранить раздел и выйти из утилиты, поэтому указываем ключ w и нажимаем Enter.

w Enter


Для проверки воспользуемся все той же командой

fdisk –l

[root@postgre vorori]# fdisk -l
WARNING: fdisk GPT support is currently new, and therefore in an experimental phase. Use at your own discretion.

\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\
Disk /dev/vda: 21.5 GB, 21474836480 bytes, 41943040 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disk label type: gpt
Disk identifier: 01791D85-D081-4F33-BADF-3DE986DA6219


#         Start          End    Size  Type            Name
 1         2048         4095      1M  BIOS boot
 2         4096     41943005     20G  Microsoft basic
\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\


вот наш диск 10gb!!!!
//////////////////////////////////////////////////////////////////////////////////////
Disk /dev/vdb: 10.7 GB, 10737418240 bytes, 20971520 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 8192 bytes
I/O size (minimum/optimal): 8192 bytes / 8192 bytes
Disk label type: dos
Disk identifier: 0x0be8e770

   Device Boot      Start         End      Blocks   Id  System
/dev/vdb1            2048    20971519    10484736   83  Linux
//////////////////////////////////////////////////////////////////////////////////////



7.4
пред выполнение данного пункта 7.5 установить 
yum install lvm2

lsblk
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
vda    253:0    0  20G  0 disk
├─vda1 253:1    0   1M  0 part
└─vda2 253:2    0  20G  0 part /
vdb    253:16   0  10G  0 disk
└─vdb1 253:17   0  10G  0 part

7.5
Теперь создадим физический том (physical volume), 
для этого воспользуемся утилитой pvcreate и укажем имя нашего нового раздела:

pvcreate /dev/vdb1 
[root@postgre vorori]# pvcreate /dev/vdb1
  Physical volume "/dev/vdb1" successfully created.

Для проверки воспользуемся командой pvdisplay.

[root@postgre vorori]# pvdisplay
  "/dev/vdb1" is a new physical volume of "<10.00 GiB"
   --- NEW Physical volume ---
  PV Name               /dev/vdb1
  VG Name
  PV Size               <10.00 GiB
  Allocatable           NO
  PE Size               0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               a0Z11V-y6M4-Z2fm-dTZ2-ZpsZ-pJZ2-o2DuV1


7.6
Добавим новую группу томов (VolumeGroupName), воспользуемся утилитой vgcreate, 
а группу томов назовем vol и укажем через пробел, физический том. 
Если томов несколько, то их так же разделяем пробелом.

vgcreate vol /dev/vdb1

[root@postgre vorori]# vgcreate vol /dev/vdb1
  Volume group "vol" successfully created

7.7
Для проверки воспользуемся командой vgdisplay.

[root@postgre vorori]# vgdisplay
  --- Volume group ---
  VG Name               vol
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <10.00 GiB
  PE Size               4.00 MiB
  Total PE              2559
  Alloc PE / Size       0 / 0
  Free  PE / Size       2559 / <10.00 GiB
  VG UUID               KtRgS8-UZOq-wYgY-jwSg-Llab-fr5j-Bpx3O0


Как видим у нас появилась новая группа томов, с нашим физическим томом.

[root@postgre vorori]# sudo lsblk --fs
NAME   FSTYPE      LABEL UUID                                   MOUNTPOINT
vda
├─vda1
└─vda2 xfs               73c7a5bb-d81e-4815-8bef-f7b261426bab   /
vdb
└─vdb1 LVM2_member       a0Z11V-y6M4-Z2fm-dTZ2-ZpsZ-pJZ2-o2DuV1


[root@postgre vorori]# sudo lsblk -o NAME,FSTYPE,LABEL,UUID,MOUNTPOINT
NAME   FSTYPE      LABEL UUID                                   MOUNTPOINT
vda
├─vda1
└─vda2 xfs               73c7a5bb-d81e-4815-8bef-f7b261426bab   /
vdb
└─vdb1 LVM2_member       a0Z11V-y6M4-Z2fm-dTZ2-ZpsZ-pJZ2-o2DuV1



7.8
Теперь нам осталось создать логический том в существующей группе томов, для этого предназначена 
утилита lvcreate, чтобы создать логический том, максимального раздела команда будет выглядеть так:

lvcreate -l+100%FREE vol
[root@postgre vorori]# lvcreate -l+100%FREE vol
  Logical volume "lvol0" created.


Для проверки, так же существует команда lvdisplay

[root@postgre vorori]# lvdisplay
  --- Logical volume ---
  LV Path                /dev/vol/lvol0
  LV Name                lvol0
  VG Name                vol
  LV UUID                uTbdRS-TfJe-IbnW-VTSw-qefS-XiTp-06rLIo
  LV Write Access        read/write
  LV Creation host, time postgre.ru-central1.internal, 2023-05-15 13:05:53 +0000
  LV Status              available
  # open                 0
  LV Size                <10.00 GiB
  Current LE             2559
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           252:0



[root@postgre vorori]# lsblk
NAME          MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
vda           253:0    0  20G  0 disk
├─vda1        253:1    0   1M  0 part
└─vda2        253:2    0  20G  0 part /
vdb           253:16   0  10G  0 disk
└─vdb1        253:17   0  10G  0 part
  └─vol-lvol0 252:0    0  10G  0 lvm


  [root@postgre vorori]# sudo lsblk -o NAME,FSTYPE,LABEL,UUID,MOUNTPOINT
NAME          FSTYPE      LABEL UUID                                   MOUNTPOINT
vda
├─vda1
└─vda2        xfs               73c7a5bb-d81e-4815-8bef-f7b261426bab   /
vdb
└─vdb1        LVM2_member       a0Z11V-y6M4-Z2fm-dTZ2-ZpsZ-pJZ2-o2DuV1
  └─vol-lvol0


7.9
Имя нового раздела будет lvol0. Чтобы избежать путаницы изменим имя нашего раздела на newpostgre, применив команду:

 lvrename /dev/vol/lvol0 /dev/vol/newpostgre 

[root@postgre vorori]# lvrename /dev/vol/lvol0 /dev/vol/newpostgre
  Renamed "lvol0" to "newpostgre" in volume group "vol"

И проверим, что имя изменилось.
[root@postgre vorori]# lvdisplay
  --- Logical volume ---
  LV Path                /dev/vol/newpostgre
  LV Name                newpostgre
  VG Name                vol
  LV UUID                uTbdRS-TfJe-IbnW-VTSw-qefS-XiTp-06rLIo
  LV Write Access        read/write
  LV Creation host, time postgre.ru-central1.internal, 2023-05-15 13:05:53 +0000
  LV Status              available
  # open                 0
  LV Size                <10.00 GiB
  Current LE             2559
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           252:0


7.10
Отформатируем наш логический том, форматировать будем в файловую систему xfs. 
Для этого выполним следующую команду.

mkfs.xfs –f /dev/vol/newpostgre

[root@postgre vorori]# mkfs.xfs -f /dev/vol/newpostgre
specified blocksize 4096 is less than device physical sector size 8192
switching to logical sector size 512
meta-data=/dev/vol/newpostgre    isize=512    agcount=4, agsize=655104 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=2620416, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0


По окончании появится информацию о отформатированном разделе, размере блоков и прочее.


7.11
Теперь давайте в корневом разделе создадим новую папку newpostgresss, к которой и примонтируем данный раздел.

sudo mkdir -p /newpostgresss


7.12
теперь примонтируем раздел к папке. За монтирование отвечает команда mount, которой необходимо указать ключи и тип операций с монтируемым разделом. Данный логический раздел мы будем монтировать на чтение и запись, поэтому укажем ключи rw. В общем случае команда будет выглядеть так:

mount –o,rw /dev/vol/newpostgre /newpostgresss
mount –o,rw /dev/vol/newpostgre /newpostgresss
mount –o,rw /dev/vol/newpostgre /newpostgresss


проверяем

[root@postgre /]# df -h
Filesystem                  Size  Used Avail Use% Mounted on
devtmpfs                    894M     0  894M   0% /dev
tmpfs                       919M  1.1M  918M   1% /dev/shm
tmpfs                       919M  604K  919M   1% /run
tmpfs                       919M     0  919M   0% /sys/fs/cgroup
/dev/vda2                    20G  1.9G   19G  10% /
tmpfs                       184M     0  184M   0% /run/user/1000
/dev/mapper/vol-newpostgre   10G   33M   10G   1% /newpostgresss



7.13
К сожалению операция монтирования действует, только до перезагрузки для того чтобы наш диск остался примонтирован даже после перезагрузки, нам необходимо внести правки в файл fstab в каталоге etc.

Поэтому откроем файл:

vim /etc/fstab
И добавим следующую строку:

/dev/vol/newpostgre          /newpostgresss                   xfs     defaults        0 0


Далее сохраним файл и перезагрузим систему (выполним команду reboot), после чего вновь проверим, что наш логический том примонтирован к каталогу.

reboot -h now


</pre>


#### 8)

####  проинициализируйте диск согласно инструкции и подмонтировать файловую систему, только не забывайте менять имя диска на актуальное, 
####  в вашем случае это скорее всего будет /dev/sdb - https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux

<pre>
готово
</pre>


#### 9)

####  перезагрузите инстанс и убедитесь, что диск остается примонтированным (если не так смотрим в сторону fstab)

<pre>
готово
</pre>



#### 10)

####   сделайте пользователя postgres владельцем /mnt/data - chown -R postgres:postgres /newpostgresss

<pre>
chown -R postgres:postgres /newpostgresss
</pre>



#### 11)

####   перенесите содержимое /var/lib/postgresql/15 в /mnt/data - mv /var/lib/postgresql/15 /mnt/data

<pre>
systemctl stop postgresql-15.service
mv /var/lib/pgsql/15 /newpostgresss
</pre>



#### 12)

####   попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 15 main start

<pre>
systemctl start postgresql-15.service

</pre>





#### 13)

####   напишите получилось или нет и почему

<pre>

Кластер не стартует потому что мы преместили каталог с данными на другой диск в другую директорию
</pre>



#### 14)

####   попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 15 main start 
####   напишите получилось или нет и почему

<pre>
vim /usr/lib/systemd/system/postgresql-15.service
меняю значение пременной
Environment=PGDATA=/newpostgresss/15/data


обязательно конфигурацию демона systemctl
systemctl daemon-reload  (презапускаю конфигурацию демона systemctl )
</pre>



#### 15)

####   задание: найти конфигурационный параметр в файлах раположенных в /etc/postgresql/14/main который надо поменять и поменяйте его
####   напишите что и почему поменяли


<pre>

напишите что и почему поменяли
менял пременную Environment в службе postgresql-15.service которая отвечает за местоположения каталога с данными postgres
</pre>



#### 16)

####   попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 15 main start
####   напишите получилось или нет и почему
<pre>

все работает все получилось postgre сейчас работает на другом диске и служба обрашается на новый каталог с данными на новом диске

</pre>



#### 17)

####   зайдите через через psql и проверьте содержимое ранее созданной таблицы

<pre>
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

</pre>


#### 18)

####   задание со звездочкой *: не удаляя существующий GCE инстанс/ЯО сделайте новый, поставьте на его PostgreSQL,
####   удалите файлы с данными из /var/lib/postgresql, перемонтируйте внешний диск который сделали ранее 
####   от первой виртуальной машины ко второй и запустите PostgreSQL на второй машине так чтобы он работал с данными 
####   на внешнем диске, расскажите как вы это сделали и что в итоге получилось.

<pre>

естественно пред тем как отмонтировать диск с первой виртуалки остановил службу postgres на первром сервере чтобы не повредить данные и выключаю сервер

Все получилось  кластер стартанул с примапленного диска с данными со старой виртуалки. 
сделал все тоже самое только диск не надо форматировать существующий диск а монтировать с данными и тогда все ок



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
vim /usr/lib/systemd/system/postgresql-15.service

меняю значение пременной
Environment=PGDATA=/mnt/data/15/data

обязательно конфигурацию демона systemctl
systemctl daemon-reload  (презапускаю конфигурацию демона systemctl )

systemctl start postgresql-15
● postgresql-15.service - PostgreSQL 15 database server
   Loaded: loaded (/usr/lib/systemd/system/postgresql-15.service; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2023-05-15 14:04:04 UTC; 12s ago
</pre>