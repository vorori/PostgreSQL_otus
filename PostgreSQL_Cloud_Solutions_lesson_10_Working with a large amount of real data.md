#### 0)

#### ссылки на почитать

<pre>
---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
-------------------------------------------------мануалы на изучение-------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
</pre>

<pre>
https://www.youtube.com/watch?v=-T3zl7aE0Yk
https://www.youtube.com/watch?v=-T3zl7aE0Yk
https://www.youtube.com/watch?v=-T3zl7aE0Yk
https://www.youtube.com/watch?v=-T3zl7aE0Yk

https://github.com/greenplum-db/gpdb/releases
https://github.com/greenplum-db/gpdb/releases
https://github.com/greenplum-db/gpdb/releases


http://dbaselife.com/doc/1636/
http://dbaselife.com/doc/1636/
http://dbaselife.com/doc/1636/


https://www.youtube.com/watch?v=qpwA1z3s4uA
https://www.youtube.com/watch?v=qpwA1z3s4uA
https://www.youtube.com/watch?v=qpwA1z3s4uA


https://docs.vmware.com/en/VMware-Greenplum/7/greenplum-database/admin_guide-managing-startstop.html
https://docs.vmware.com/en/VMware-Greenplum/7/greenplum-database/admin_guide-managing-startstop.html
https://docs.vmware.com/en/VMware-Greenplum/7/greenplum-database/admin_guide-managing-startstop.html


создание таблиц и распределение данных в gp
Объявление ключей распределения в Greenplum 
http://www.dbaref.com/declaring-distribution-keys-in-greenplum
http://www.dbaref.com/declaring-distribution-keys-in-greenplum
http://www.dbaref.com/declaring-distribution-keys-in-greenplum


на почитать для gp чтобы разобраться в терменалогии
https://docs.vmware.com/en/VMware-Greenplum/7/greenplum-database/admin_guide-intro-arch_overview.html
https://docs.vmware.com/en/VMware-Greenplum/7/greenplum-database/admin_guide-intro-arch_overview.html
https://docs.vmware.com/en/VMware-Greenplum/7/greenplum-database/admin_guide-intro-arch_overview.html

https://github.com/greenplum-db/gpdb/wiki/Greenplum-Architecture
https://github.com/greenplum-db/gpdb/wiki/Greenplum-Architecture
https://github.com/greenplum-db/gpdb/wiki/Greenplum-Architecture

https://bigdataschool.ru/blog/greenplum-architecture.html
https://bigdataschool.ru/blog/greenplum-architecture.html
https://bigdataschool.ru/blog/greenplum-architecture.html


запуск gp 
gpstart
gpstart
gpstart

gpstop
gpstop
gpstop

</pre>




#### 1)

#### Устанаввливаю настраиваю Greenplum

<pre>
---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
----------------------------------мануал создания для двух нод шесть сегментов  START--------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
</pre>


#### подготовительные мероприятия 
<pre>
1.1) 

gp1 130.193.55.52      ssh vorori@130.193.55.52    gp1.ru-central1.internal
gp2 158.160.22.211     ssh vorori@158.160.22.211   gp2.ru-central1.internal

sudo yum -y install epel-release
yum install htop
yum install mc
yum install vim
yum install wget

vim /etc/selinux/config
set disabled
sestatus
SELinux status:                 disabled
reboot -h now
</pre>


#### общаая инфа по ресурсам
<pre>
1.2)

cat /etc/redhat-release
cat /proc/cpuinfo | grep processor
cat /proc/meminfo  | grep MemTotal
</pre>



#### настройка ОС
<pre>
1.3)

vim /etc/sysctl.conf
vm.overcommit_memory = 2
vm.overcommit_ratio = 95 
net.ipv4.ip_local_port_range = 10000 65535 
kernel.sem = 250 2048000 200 8192
sysctl --system
</pre>



#### создать столько файлов и процессов вплодь до этого числа
<pre>
1.4)

vim /etc/security/limits.conf 
* soft nofile 524288
* hard nofile 524288
* soft nproc 131072
* hard nproc 131072

</pre>


#### добавляем групу gpadmin
<pre>
1.5)

groupadd gpadmin
useradd gpadmin -r -m -g gpadmin
echo Q123456789q | passwd --stdin gpadmin
</pre>

#### открываю порты если разворачиваемся не на яндекс облаке
<pre>
1.6)

firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="10.129.0.13" port protocol="tcp" port="22" accept"
firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="10.129.0.31" port protocol="tcp" port="22" accept"
firewall-cmd --reload		
firewall-cmd --list-all		
</pre>

#### скачиваем устанавливаем
<pre>
1.7)

wget https://github.com/greenplum-db/gpdb/releases/download/6.24.3/open-source-greenplum-db-6.24.3-rhel7-x86_64.rpm
yum install ./open-source-greenplum-db-6.24.3-rhel7-x86_64.rpm


===================================================================================================================
 Package                       Arch      Version            Repository                                        Size
===================================================================================================================
Installing:
 open-source-greenplum-db-6    x86_64    6.24.3-1.el7       /open-source-greenplum-db-6.24.3-rhel7-x86_64    295 M
Installing for dependencies:
 apr                           x86_64    1.4.8-7.el7        base                                             104 k
 apr-util                      x86_64    1.5.2-6.el7_9.1    updates                                           92 k
 keyutils-libs-devel           x86_64    1.5.8-3.el7        base                                              37 k
 krb5-devel                    x86_64    1.15.1-55.el7_9    updates                                          273 k
 libcgroup                     x86_64    0.41-21.el7        base                                              66 k
 libcgroup-tools               x86_64    0.41-21.el7        base                                              99 k
 libcom_err-devel              x86_64    1.42.9-19.el7      base                                              32 k
 libevent                      x86_64    2.0.21-4.el7       base                                             214 k
 libkadm5                      x86_64    1.15.1-55.el7_9    updates                                          180 k
 libselinux-devel              x86_64    2.5-15.el7         base                                             187 k
 libsepol-devel                x86_64    2.5-10.el7         base                                              77 k
 libverto-devel                x86_64    0.2.5-4.el7        base                                              12 k
 pcre-devel                    x86_64    8.32-17.el7        base                                             480 k
 rsync                         x86_64    3.1.2-12.el7_9     updates                                          408 k
 zip                           x86_64    3.0-11.el7         base                                             260 k

Transaction Summary
===================================================================================================================
Install  1 Package (+15 Dependent packages)

Total size: 298 M
Total download size: 2.5 M
Installed size: 301 M
Is this ok [y/d/N]: y
</pre>

#### проверяем
<pre>
1.8)

cd /usr/local
ls -l

total 0
drwxr-xr-x.  2 root root   6 Apr 11  2018 bin
drwxr-xr-x.  2 root root   6 Apr 11  2018 etc
drwxr-xr-x.  2 root root   6 Apr 11  2018 games
lrwxrwxrwx   1 root root  30 Jun  9 22:24 greenplum-db -> /usr/local/greenplum-db-6.24.3
drwxr-xr-x  11 root root 238 Jun  9 22:24 greenplum-db-6.24.3
drwxr-xr-x.  2 root root   6 Apr 11  2018 include
drwxr-xr-x.  2 root root   6 Apr 11  2018 lib
drwxr-xr-x.  2 root root   6 Apr 11  2018 lib64
drwxr-xr-x.  2 root root   6 Apr 11  2018 libexec
drwxr-xr-x.  8 root root 214 Jun  9 22:15 okagent
drwxr-xr-x.  2 root root   6 Apr 11  2018 sbin
drwxr-xr-x.  5 root root  49 Feb 11  2020 share
drwxr-xr-x.  2 root root   6 Apr 11  2018 src
</pre>

####  заходим под пользователем gpadmin для установки настройки
<pre>
1.9)

su - gpadmin

настравиваем среду к файлу greenplum_path.sh
vim /home/gpadmin/.bashrc

# User specific aliases and functions
. /usr/local/greenplum-db/greenplum_path.sh


[gpadmin@client8 ~]$ exit
logout
su - gpadmin

[gpadmin@client8 ~]$ cd $GPHOME/
[gpadmin@client8 greenplum-db-6.24.3]$
[gpadmin@client8 greenplum-db-6.24.3]$ which psql
/usr/local/greenplum-db-6.24.3/bin/psql
</pre>

#### Создайте файл с именем hosts в домашнем каталоге пользователя gpadmin, который содержит все имена хостов Greenplum
<pre>
1.10)

vim /home/gpadmin/.hosts
vim /home/gpadmin/.hosts
vim /home/gpadmin/.hosts

hostname
gp1.ru-central1.internal
gp2.ru-central1.internal
</pre>

#### заходим под пользователем gpadmin для установки настройки
<pre>
1.11)

su - gpadmin

настравиваем среду к файлу greenplum_path.sh
vim /home/gpadmin/.bashrc

вставляем этот кусок
-------------------------------------------------------------------------------------------
# User specific aliases and functions
. /usr/local/greenplum-db/greenplum_path.sh
-------------------------------------------------------------------------------------------

exit
logout
su - gpadmin
</pre>

####  проверяем $GPHOME
<pre>
1.12)

cd $GPHOME/
which psql
/usr/local/greenplum-db-6.24.3/bin/psql
</pre>

#### генерирую ключ
<pre>
1.12.1)

Выполните следующие шаги на главном хосте в качестве пользователя gpadmin.
генерирую ключ (на второй ноде тоже чтобы создать каталог ./ssh)
su - gpadmin
[gpadmin@client8 ~]$ ssh-keygen -t rsa
                     ssh-keygen -t rsa -b 4096
                      
Generating public/private rsa key pair.
Enter file in which to save the key (/home/gpadmin/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/gpadmin/.ssh/id_rsa.
Your public key has been saved in /home/gpadmin/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:nGlzyL4QR3txb9Ynr6W6h2AuVAspFAng+59VtuzjNb8 gpadmin@client8.unixmen.local
The key's randomart image is:
+---[RSA 2048]----+
|  .....o         |
| .    o          |
|  .  .  ... .    |
|   .  .+o=.o . . |
|  .   ..So=.  = o|
|   .   =.B+. o o.|
|    . ..ooo.o.  o|
|     . +.oo..o.+ |
|      o .oo.o+E. |
+----[SHA256]-----+
</pre>

#### передаю публичный ключ на сервер ноду 2
1.12.2)
<pre>
#после генерации мне надо предать этот публичный ключ на сервер ноду 2
ssh-copy-id -i ~/.ssh/id_rsa.pub gpadmin@gp2.ru-central1.internal -p 22
ssh-copy-id -i ~/.ssh/id_rsa.pub gpadmin@gp2.ru-central1.internal -p 22
ssh-copy-id -i ~/.ssh/id_rsa.pub gpadmin@gp2.ru-central1.internal -p 22

#проверяем что можем подключиться
ssh gpadmin@gp2.ru-central1.internal
ssh gpadmin@gp2.ru-central1.internal
ssh gpadmin@gp2.ru-central1.internal
</pre>


#### если по старинке
<pre>
--------------------------------------------------------------------------
1.12.3)

если по старинке создал на сервере к которому буду подключаться
создал файл
touch /home/gpadmin/.ssh/authorized_keys
touch /home/gpadmin/.ssh/authorized_keys
touch /home/gpadmin/.ssh/authorized_keys

#### выполняю
1.12.4)

вставил в него мой публичный ключ с гланого сервера 
cat /home/gpadmin/.ssh/id_rsa.pub                # копирую публичный ключ пользователя gpadmin сервера с которого буду подключаться
vim /home/gpadmin/.ssh/authorized_keys           # вставляю
chmod 600 /home/gpadmin/.ssh/authorized_keys     # выдаю правельные разрешения
--------------------------------------------------------------------------
</pre>

#### Утилита gpssh-exkeys
<pre>
1.13)

Утилита gpssh-exkeys обменивается ключами SSH между указанными именами хостов (или адресами хостов). 
Это позволяет подключаться по SSH между хостами Greenplum и сетевыми интерфейсами без запроса пароля. 
Утилита используется для первоначальной подготовки системы базы данных Greenplum для беспарольного доступа по SSH, 
а также для подготовки дополнительных хостов для беспарольного доступа по SSH при расширении системы базы данных Greenplum.
https://docs.vmware.com/en/VMware-Greenplum/6/greenplum-database/utility_guide-ref-gpssh-exkeys.html
-----------------------------------------------------------------------------------------------------------------------------------------------------
ВАЖНО ЗАМЕТКА:!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
чтобы это ЗАРАБОТАЛО НУЖНО СНАЧАЛО ПОДКЛЮЧИТЬСЯ ВРУЧНУЮ!!!!!!!
иначе будем получать ошибку такого плана 
[ERROR]: Failed to ssh to gpadmin@gp2.ru-central1.internal. No ECDSA host key is known for gpadmin@gp2.ru-central1.internal and you have requested strict checking.
Host key verification failed.

подключаемся к ноде по ssh gpadmin@gp2.ru-central1.internal
ssh gpadmin@gp2.ru-central1.internal
ssh gpadmin@gp2.ru-central1.internal
ssh gpadmin@gp2.ru-central1.internal
-----------------------------------------------------------------------------------------------------------------------------------------------------


выполняем на главном gpadmin@gp1.ru-central1.internal
gpssh-exkeys -f /home/gpadmin/.hosts
gpssh-exkeys -f /home/gpadmin/.hosts
gpssh-exkeys -f /home/gpadmin/.hosts


лучше включить лог на второстипенном
tail -f /var/log/messages
tail -f /var/log/messages
tail -f /var/log/messages

ВНИМАТЕЛЬНО! !!!!! смотрим что все прошло успешно без ошибок!!!!!!!!!!!!!!!!!!!
gpssh-exkeys -f /home/gpadmin/.hosts

[STEP 1 of 5] create local ID and authorize on local host
  ... /home/gpadmin/.ssh/id_rsa file exists ... key generation skipped

[STEP 2 of 5] keyscan all hosts and update known_hosts file

[STEP 3 of 5] retrieving credentials from remote hosts
  ... send to gp2.ru-central1.internal

[STEP 4 of 5] determine common authentication file content

[STEP 5 of 5] copy authentication files to all remote hosts
  ... finished key exchange with gp2.ru-central1.internal

[INFO] completed successfully

</pre>
	 
#### проверяю ВСЕ КЛЮЧИ
<pre>
1.13.1)

на сервере ноде ДОЛДЖНЫ ПОЯВИТЬСЯ ВСЕ КЛЮЧИ!
появятся вот эти файлы причем и на ноде которую мы подключили
ls -l /home/gpadmin/.ssh

[gpadmin@gp2 ~]$ ls -l /home/gpadmin/.ssh
total 16
-rw------- 1 gpadmin gpadmin  758 Jun 12 18:11 authorized_keys
-rw-r--r-- 1 gpadmin gpadmin    0 Jun 12 18:11 config
-rw-r--r-- 1 gpadmin gpadmin    0 Jun 12 18:11 iddummy.pub
-rw------- 1 gpadmin gpadmin 3243 Jun 12 18:11 id_rsa
-rw-r--r-- 1 gpadmin gpadmin  758 Jun 12 18:11 id_rsa.pub
-rw-rw-r-- 1 gpadmin gpadmin 1403 Jun 12 18:11 known_hosts
</pre>

#### создаю директорию для будующих данных
<pre>
1.13.2)

создаю директорию для будующих данных
Утилита gpsshпозволяет запускать команды оболочки bash на нескольких хостах одновременно, используя SSH (защищенную оболочку). 
Вы можете запустить одну команду, указав ее в командной строке, или пропустить команду, чтобы войти в интерактивный сеанс командной строки.
создаю первичные и зеркальные каталоги данных на всех хостах сегмента.
su - gpadmin
gpssh -f /home/gpadmin/.hosts
mkdir /home/gpadmin/data
mkdir /home/gpadmin/data_mirror

[gpadmin@gp1 ~]$ gpssh -f /home/gpadmin/.hosts
=> mkdir /home/gpadmin/data
[gp1.ru-central1.internal]
[gp2.ru-central1.internal]
=> mkdir /home/gpadmin/data_mirror
[gp1.ru-central1.internal]
[gp2.ru-central1.internal]


на ноде и на мастере появятся каталоги
[gpadmin@gp2 ~]$ ls -l /home/gpadmin
total 0
drwxrwxr-x 2 gpadmin gpadmin 6 Jun 12 18:16 data
drwxrwxr-x 2 gpadmin gpadmin 6 Jun 12 18:16 data_mirror
</pre>

#### настройка конфигурационного файла на главной ноде
<pre>
1.14)

настройка конфигурационного файла на главной ноде смотрим что у нас есть в наличии
cd $GPHOME/docs/cli_help/gpconfigs
ls -l
total 52
-rw-r--r-- 1 root root 2422 May  5 00:13 gpinitsystem_config
-rw-r--r-- 1 root root 4511 May  5 00:13 gpinitsystem_singlenode
-rw-r--r-- 1 root root 2321 May  5 00:13 gpinitsystem_test
-rw-r--r-- 1 root root  359 May  5 00:13 hostfile_exkeys
-rw-r--r-- 1 root root  119 May  5 00:13 hostfile_gpchecknet_ic1
-rw-r--r-- 1 root root  119 May  5 00:13 hostfile_gpchecknet_ic2
-rw-r--r-- 1 root root   87 May  5 00:13 hostfile_gpcheckperf
-rw-r--r-- 1 root root  255 May  5 00:13 hostfile_gpexpand
-rw-r--r-- 1 root root  237 May  5 00:13 hostfile_gpinitsystem
-rw-r--r-- 1 root root   96 May  5 00:13 hostfile_gpssh_allhosts
-rw-r--r-- 1 root root   87 May  5 00:13 hostfile_gpssh_segonly
-rw-r--r-- 1 root root   44 May  5 00:13 hostlist_singlenode


копирую конфиг для создания двух в домашний каталог
cp $GPHOME/docs/cli_help/gpconfigs/gpinitsystem_config /home/gpadmin/gpinitsystem_config
cp $GPHOME/docs/cli_help/gpconfigs/gpinitsystem_config /home/gpadmin/gpinitsystem_config
cp $GPHOME/docs/cli_help/gpconfigs/gpinitsystem_config /home/gpadmin/gpinitsystem_config

редактирую настройки
vim /home/gpadmin/gpinitsystem_config

----------------------------------------------------------------------------------------------------------------------------------------------------------------
описание параметров:

указываю мой файл с хостами
MACHINE_LIST_FILE=/home/gpadmin/.hosts

Местоположение файловой системы, в котором будет создан основной каталог данных сегмента. 
Количество расположений в списке определяет количество основных сегментов, которые будут созданы для каждого физического 
хоста (если в файле hosts указано несколько адресов для хоста, количество сегментов будет равномерно распределено по указанным адресам интерфейсов).
declare -a DATA_DIRECTORY=(/home/gpadmin/data /home/gpadmin/data /home/gpadmin/data)


вписываю доменное имя моей главной машины
MASTER_HOSTNAME=gp1.ru-central1.internal

а также каталог с основными данными будет находится где и рабочие файлы gp
MASTER_DIRECTORY=/home/gpadmin/data


Местоположение файловой системы, в котором будет создан каталог данных сегмента зеркало. 
Количество зеркальных расположений должно быть равно количеству первичных расположений, указанному в параметре DATA_DIRECTORY.
declare -a MIRROR_DATA_DIRECTORY=(/home/gpadmin/data_mirror /home/gpadmin/data_mirror /home/gpadmin/data_mirror)
----------------------------------------------------------------------------------------------------------------------------------------------------------------
</pre>

#### инициализируем кластер создаем рабочуую среду
<pre>
1.15)

инициализируем кластер создаем рабочуую среду
gpinitsystem -c /home/gpadmin/gpinitsystem_config
gpinitsystem -c /home/gpadmin/gpinitsystem_config
gpinitsystem -c /home/gpadmin/gpinitsystem_config

gpinitsystem -c /home/gpadmin/gpinitsystem_config -s gp1.ru-central1.internal
gpinitsystem -c /home/gpadmin/gpinitsystem_config -s gp1.ru-central1.internal
gpinitsystem -c /home/gpadmin/gpinitsystem_config -s gp1.ru-central1.internal

[gpadmin@gp1 gpconfigs]$ [gpadmin@gp1 gpconfigs]$ gpinitsystem -c /home/gpadmin/gpinitsystem_config -s gp1.ru-central1.internal
20230612:18:27:50:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Checking configuration parameters, please wait...
20230612:18:27:50:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Reading Greenplum configuration file /home/gpadmin/gpinitsystem_config
20230612:18:27:50:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Locale has not been set in /home/gpadmin/gpinitsystem_config, will set to default value
20230612:18:27:50:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Locale set to en_US.utf8
20230612:18:27:50:002464 gpinitsystem:gp1:gpadmin-[INFO]:-No DATABASE_NAME set, will exit following template1 updates
20230612:18:27:50:002464 gpinitsystem:gp1:gpadmin-[INFO]:-MASTER_MAX_CONNECT not set, will set to default value 250
20230612:18:27:50:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Checking configuration parameters, Completed
20230612:18:27:50:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Commencing multi-home checks, please wait...
..
20230612:18:27:51:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Configuring build for standard array
20230612:18:27:51:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Commencing multi-home checks, Completed
20230612:18:27:51:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Building primary segment instance array, please wait...
......
20230612:18:27:53:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Checking Master host
20230612:18:27:53:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Checking new segment hosts, please wait...
......
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Checking new segment hosts, Completed
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Greenplum Database Creation Parameters
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:---------------------------------------
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Master Configuration
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:---------------------------------------
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Master instance name       = Greenplum Data Platform
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Master hostname            = gp1.ru-central1.internal
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Master port                = 5432
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Master instance dir        = /home/gpadmin/data/gpseg-1
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Master LOCALE              = en_US.utf8
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Greenplum segment prefix   = gpseg
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Master Database            =
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Master connections         = 250
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Master buffers             = 128000kB
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Segment connections        = 750
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Segment buffers            = 128000kB
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Checkpoint segments        = 8
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Encoding                   = UNICODE
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Postgres param file        = Off
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Initdb to be used          = /usr/local/greenplum-db-6.24.3/bin/initdb
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:-GP_LIBRARY_PATH is         = /usr/local/greenplum-db-6.24.3/lib
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:-HEAP_CHECKSUM is           = on
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:-HBA_HOSTNAMES is           = 0
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Ulimit check               = Passed
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Array host connect type    = Single hostname per node
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Master IP address [1]      = ::1
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Master IP address [2]      = 10.129.0.31
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Master IP address [3]      = fe80::d20d:1eff:fe6f:e
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Standby Master             = gp1.ru-central1.internal
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Number of primary segments = 3
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Standby IP address         = ::1
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Standby IP address         = 10.129.0.31
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Standby IP address         = fe80::d20d:1eff:fe6f:e
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Total Database segments    = 6
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Trusted shell              = ssh
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Number segment hosts       = 2
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Mirroring config           = OFF
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:----------------------------------------
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Greenplum Primary Segment Configuration
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:----------------------------------------
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:-gp1.ru-central1.internal      6000    gp1.ru-central1.internal                                                    /home/gpadmin/data/gpseg0        2
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:-gp1.ru-central1.internal      6001    gp1.ru-central1.internal                                                    /home/gpadmin/data/gpseg1        3
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:-gp1.ru-central1.internal      6002    gp1.ru-central1.internal                                                    /home/gpadmin/data/gpseg2        4
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:-gp2.ru-central1.internal      6000    gp2.ru-central1.internal                                                    /home/gpadmin/data/gpseg3        5
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:-gp2.ru-central1.internal      6001    gp2.ru-central1.internal                                                    /home/gpadmin/data/gpseg4        6
20230612:18:27:59:002464 gpinitsystem:gp1:gpadmin-[INFO]:-gp2.ru-central1.internal      6002    gp2.ru-central1.internal                                                    /home/gpadmin/data/gpseg5        7

Continue with Greenplum creation Yy|Nn (default=N):
> y
20230612:18:28:13:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Building the Master instance database, please wait...
20230612:18:28:19:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Starting the Master in admin mode
20230612:18:28:19:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Commencing parallel build of primary segment instances
20230612:18:28:19:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Spawning parallel processes    batch [1], please wait...
......
20230612:18:28:19:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Waiting for parallel processes batch [1], please wait...
....................
20230612:18:28:40:002464 gpinitsystem:gp1:gpadmin-[INFO]:------------------------------------------------
20230612:18:28:40:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Parallel process exit status
20230612:18:28:40:002464 gpinitsystem:gp1:gpadmin-[INFO]:------------------------------------------------
20230612:18:28:40:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Total processes marked as completed           = 6
20230612:18:28:40:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Total processes marked as killed              = 0
20230612:18:28:40:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Total processes marked as failed              = 0
20230612:18:28:40:002464 gpinitsystem:gp1:gpadmin-[INFO]:------------------------------------------------
20230612:18:28:40:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Removing back out file
20230612:18:28:40:002464 gpinitsystem:gp1:gpadmin-[INFO]:-No errors generated from parallel processes
20230612:18:28:40:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Restarting the Greenplum instance in production mode
20230612:18:28:40:009438 gpstop:gp1:gpadmin-[INFO]:-Starting gpstop with args: -a -l /home/gpadmin/gpAdminLogs -m -d /home/gpadmin/data/gpseg-1
20230612:18:28:40:009438 gpstop:gp1:gpadmin-[INFO]:-Gathering information and validating the environment...
20230612:18:28:40:009438 gpstop:gp1:gpadmin-[INFO]:-Obtaining Greenplum Master catalog information
20230612:18:28:40:009438 gpstop:gp1:gpadmin-[INFO]:-Obtaining Segment details from master...
20230612:18:28:40:009438 gpstop:gp1:gpadmin-[INFO]:-Greenplum Version: 'postgres (Greenplum Database) 6.24.3 build commit:25d3498a400ca5230e81abb94861f23389315213 Open Source'
20230612:18:28:40:009438 gpstop:gp1:gpadmin-[INFO]:-Commencing Master instance shutdown with mode='smart'
20230612:18:28:40:009438 gpstop:gp1:gpadmin-[INFO]:-Master segment instance directory=/home/gpadmin/data/gpseg-1
20230612:18:28:41:009438 gpstop:gp1:gpadmin-[INFO]:-Stopping master segment and waiting for user connections to finish ...
server shutting down
20230612:18:28:42:009438 gpstop:gp1:gpadmin-[INFO]:-Attempting forceful termination of any leftover master process
20230612:18:28:42:009438 gpstop:gp1:gpadmin-[INFO]:-Terminating processes for segment /home/gpadmin/data/gpseg-1
20230612:18:28:43:009670 gpstart:gp1:gpadmin-[INFO]:-Starting gpstart with args: -a -l /home/gpadmin/gpAdminLogs -d /home/gpadmin/data/gpseg-1
20230612:18:28:43:009670 gpstart:gp1:gpadmin-[INFO]:-Gathering information and validating the environment...
20230612:18:28:43:009670 gpstart:gp1:gpadmin-[INFO]:-Greenplum Binary Version: 'postgres (Greenplum Database) 6.24.3 build commit:25d3498a400ca5230e81abb94861f23389315213 Open Source'
20230612:18:28:43:009670 gpstart:gp1:gpadmin-[INFO]:-Greenplum Catalog Version: '301908232'
20230612:18:28:43:009670 gpstart:gp1:gpadmin-[INFO]:-Starting Master instance in admin mode
20230612:18:28:43:009670 gpstart:gp1:gpadmin-[INFO]:-Obtaining Greenplum Master catalog information
20230612:18:28:43:009670 gpstart:gp1:gpadmin-[INFO]:-Obtaining Segment details from master...
20230612:18:28:43:009670 gpstart:gp1:gpadmin-[INFO]:-Setting new master era
20230612:18:28:43:009670 gpstart:gp1:gpadmin-[INFO]:-Master Started...
20230612:18:28:43:009670 gpstart:gp1:gpadmin-[INFO]:-Shutting down master
20230612:18:28:45:009670 gpstart:gp1:gpadmin-[INFO]:-Commencing parallel segment instance startup, please wait...
...
20230612:18:28:48:009670 gpstart:gp1:gpadmin-[INFO]:-Process results...
20230612:18:28:48:009670 gpstart:gp1:gpadmin-[INFO]:-----------------------------------------------------
20230612:18:28:48:009670 gpstart:gp1:gpadmin-[INFO]:-   Successful segment starts                                            = 6
20230612:18:28:48:009670 gpstart:gp1:gpadmin-[INFO]:-   Failed segment starts                                                = 0
20230612:18:28:48:009670 gpstart:gp1:gpadmin-[INFO]:-   Skipped segment starts (segments are marked down in configuration)   = 0
20230612:18:28:48:009670 gpstart:gp1:gpadmin-[INFO]:-----------------------------------------------------
20230612:18:28:48:009670 gpstart:gp1:gpadmin-[INFO]:-Successfully started 6 of 6 segment instances
20230612:18:28:48:009670 gpstart:gp1:gpadmin-[INFO]:-----------------------------------------------------
20230612:18:28:48:009670 gpstart:gp1:gpadmin-[INFO]:-Starting Master instance gp1.ru-central1.internal directory /home/gpadmin/data/gpseg-1
20230612:18:28:50:009670 gpstart:gp1:gpadmin-[INFO]:-Command pg_ctl reports Master gp1.ru-central1.internal instance active
20230612:18:28:50:009670 gpstart:gp1:gpadmin-[INFO]:-Connecting to dbname='template1' connect_timeout=15
20230612:18:28:50:009670 gpstart:gp1:gpadmin-[INFO]:-No standby master configured.  skipping...
20230612:18:28:50:009670 gpstart:gp1:gpadmin-[INFO]:-Database successfully started
20230612:18:28:50:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Completed restart of Greenplum instance in production mode
20230612:18:28:50:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Starting initialization of standby master gp1.ru-central1.internal
20230612:18:28:51:010080 gpinitstandby:gp1:gpadmin-[INFO]:-Validating environment and parameters for standby initialization...
20230612:18:28:51:010080 gpinitstandby:gp1:gpadmin-[INFO]:-Checking for data directory /home/gpadmin/data/gpseg-1 on gp1.ru-central1.internal
20230612:18:28:52:002464 gpinitsystem:gp1:gpadmin-[WARN]:-Сomplete standby master initialization
20230612:18:28:52:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Scanning utility log file for any warning messages
20230612:18:28:52:002464 gpinitsystem:gp1:gpadmin-[WARN]:-*******************************************************
20230612:18:28:52:002464 gpinitsystem:gp1:gpadmin-[WARN]:-Scan of log file indicates that some warnings or errors
20230612:18:28:52:002464 gpinitsystem:gp1:gpadmin-[WARN]:-were generated during the array creation
20230612:18:28:52:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Please review contents of log file
20230612:18:28:52:002464 gpinitsystem:gp1:gpadmin-[INFO]:-/home/gpadmin/gpAdminLogs/gpinitsystem_20230612.log
20230612:18:28:52:002464 gpinitsystem:gp1:gpadmin-[INFO]:-To determine level of criticality
20230612:18:28:52:002464 gpinitsystem:gp1:gpadmin-[WARN]:-*******************************************************
20230612:18:28:52:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Greenplum Database instance successfully created
20230612:18:28:52:002464 gpinitsystem:gp1:gpadmin-[INFO]:-------------------------------------------------------
20230612:18:28:52:002464 gpinitsystem:gp1:gpadmin-[INFO]:-To complete the environment configuration, please
20230612:18:28:52:002464 gpinitsystem:gp1:gpadmin-[INFO]:-update gpadmin .bashrc file with the following
20230612:18:28:52:002464 gpinitsystem:gp1:gpadmin-[INFO]:-1. Ensure that the greenplum_path.sh file is sourced
20230612:18:28:52:002464 gpinitsystem:gp1:gpadmin-[INFO]:-2. Add "export MASTER_DATA_DIRECTORY=/home/gpadmin/data/gpseg-1"
20230612:18:28:52:002464 gpinitsystem:gp1:gpadmin-[INFO]:-   to access the Greenplum scripts for this instance:
20230612:18:28:52:002464 gpinitsystem:gp1:gpadmin-[INFO]:-   or, use -d /home/gpadmin/data/gpseg-1 option for the Greenplum scripts
20230612:18:28:52:002464 gpinitsystem:gp1:gpadmin-[INFO]:-   Example gpstate -d /home/gpadmin/data/gpseg-1
20230612:18:28:52:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Script log file = /home/gpadmin/gpAdminLogs/gpinitsystem_20230612.log
20230612:18:28:52:002464 gpinitsystem:gp1:gpadmin-[INFO]:-To remove instance, run gpdeletesystem utility
20230612:18:28:52:002464 gpinitsystem:gp1:gpadmin-[WARN]:-Standby Master failed to initialize
20230612:18:28:52:002464 gpinitsystem:gp1:gpadmin-[INFO]:-------------------------------------------------------
20230612:18:28:52:002464 gpinitsystem:gp1:gpadmin-[INFO]:-The Master /home/gpadmin/data/gpseg-1/pg_hba.conf post gpinitsystem
20230612:18:28:52:002464 gpinitsystem:gp1:gpadmin-[INFO]:-has been configured to allow all hosts within this new
20230612:18:28:52:002464 gpinitsystem:gp1:gpadmin-[INFO]:-array to intercommunicate. Any hosts external to this
20230612:18:28:52:002464 gpinitsystem:gp1:gpadmin-[INFO]:-new array must be explicitly added to this file
20230612:18:28:52:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Refer to the Greenplum Admin support guide which is
20230612:18:28:52:002464 gpinitsystem:gp1:gpadmin-[INFO]:-located in the /usr/local/greenplum-db-6.24.3/docs directory
20230612:18:28:52:002464 gpinitsystem:gp1:gpadmin-[INFO]:-------------------------------------------------------
20230612:18:28:52:002464 gpinitsystem:gp1:gpadmin-[WARN]:-*******************************************************
20230612:18:28:52:002464 gpinitsystem:gp1:gpadmin-[WARN]:-Cluster setup finished
20230612:18:28:52:002464 gpinitsystem:gp1:gpadmin-[INFO]:-Use gpinitstandby to create a Standby Master
20230612:18:28:52:002464 gpinitsystem:gp1:gpadmin-[WARN]:-*******************************************************
</pre>

#### проверил логи проверил директории с данными на обоих нодах успешно заполнены дефолтными данными все ок 
<pre>
1.16)

обычно ечли чтото у нас не получается надо смотреть сюда тут ворох логов в котором можно найти нашу закавырку если такая имеется 
[gpadmin@gp1 ~]$ ls -l /home/gpadmin/gpAdminLogs
total 216
-rw-rw-r-- 1 gpadmin gpadmin    539 Jun 12 18:28 backout_gpinitsystem_gpadmin_20230612_182750
-rw-rw-r-- 1 gpadmin gpadmin    806 Jun 12 18:28 gpinitstandby_20230612.log
-rw-rw-r-- 1 gpadmin gpadmin 197992 Jun 12 18:28 gpinitsystem_20230612.log
-rw-rw-r-- 1 gpadmin gpadmin   2071 Jun 12 18:28 gpsegstart.py_gp1:gpadmin_20230612.log
-rw-rw-r-- 1 gpadmin gpadmin   2575 Jun 12 18:28 gpstart_20230612.log
-rw-rw-r-- 1 gpadmin gpadmin   2265 Jun 12 18:28 gpstop_20230612.log
</pre>

#### проверка сегментов
<pre>
1.17)

проверяем на мастере что были созданы три рабочих сегмента  и один главный сегмент
[gpadmin@gp1 ~]$ ls -l /home/gpadmin/data
total 16
drwx------ 21 gpadmin gpadmin 4096 Jun 12 18:28 gpseg0
drwx------ 21 gpadmin gpadmin 4096 Jun 12 18:28 gpseg1
drwx------ 22 gpadmin gpadmin 4096 Jun 12 18:28 gpseg-1
drwx------ 21 gpadmin gpadmin 4096 Jun 12 18:28 gpseg2


проверка сегментов проверяем на ноде
[gpadmin@gp2 ~]$ ls -l /home/gpadmin/data
total 12
drwx------ 21 gpadmin gpadmin 4096 Jun 12 18:28 gpseg3
drwx------ 21 gpadmin gpadmin 4096 Jun 12 18:28 gpseg4
drwx------ 21 gpadmin gpadmin 4096 Jun 12 18:28 gpseg5
</pre>

#### описание главный сегмент
<pre>
1.18)

главный сегмент в названии будет отрицательное число в нашем случае -1
[gpadmin@gp1 ~]$ ls -l /home/gpadmin/data/gpseg-1
total 72
drwx------ 5 gpadmin gpadmin    41 Jun 12 18:28 base
drwx------ 2 gpadmin gpadmin  4096 Jun 12 18:28 global
drwxrwxr-x 5 gpadmin gpadmin    42 Jun 12 18:28 gpperfmon
-rw------- 1 gpadmin gpadmin   470 Jun 12 18:28 gpsegconfig_dump
-rw-rw-r-- 1 gpadmin gpadmin   860 Jun 12 18:28 gpssh.conf
-rw------- 1 gpadmin gpadmin    10 Jun 12 18:28 internal.auto.conf
drwx------ 2 gpadmin gpadmin    18 Jun 12 18:28 pg_clog
drwx------ 2 gpadmin gpadmin    18 Jun 12 18:28 pg_distributedlog
drwx------ 2 gpadmin gpadmin     6 Jun 12 18:28 pg_dynshmem
-rw-rw-r-- 1 gpadmin gpadmin  4815 Jun 12 18:28 pg_hba.conf
-rw------- 1 gpadmin gpadmin  1636 Jun 12 18:28 pg_ident.conf
drwx------ 2 gpadmin gpadmin   141 Jun 12 18:28 pg_log
drwx------ 4 gpadmin gpadmin    39 Jun 12 18:28 pg_logical
drwx------ 4 gpadmin gpadmin    36 Jun 12 18:28 pg_multixact
drwx------ 2 gpadmin gpadmin    18 Jun 12 18:28 pg_notify
drwx------ 2 gpadmin gpadmin     6 Jun 12 18:28 pg_replslot
drwx------ 2 gpadmin gpadmin     6 Jun 12 18:28 pg_serial
drwx------ 2 gpadmin gpadmin     6 Jun 12 18:28 pg_snapshots
drwx------ 2 gpadmin gpadmin     6 Jun 12 18:28 pg_stat
drwx------ 2 gpadmin gpadmin     6 Jun 12 18:28 pg_stat_tmp
drwx------ 2 gpadmin gpadmin    18 Jun 12 18:28 pg_subtrans
drwx------ 2 gpadmin gpadmin     6 Jun 12 18:28 pg_tblspc
drwx------ 2 gpadmin gpadmin     6 Jun 12 18:28 pg_twophase
drwx------ 2 gpadmin gpadmin     6 Jun 12 18:28 pg_utilitymodedtmredo
-rw------- 1 gpadmin gpadmin     4 Jun 12 18:28 PG_VERSION
drwx------ 3 gpadmin gpadmin    60 Jun 12 18:28 pg_xlog
-rw------- 1 gpadmin gpadmin    88 Jun 12 18:28 postgresql.auto.conf
-rw------- 1 gpadmin gpadmin 23645 Jun 12 18:28 postgresql.conf
-rw------- 1 gpadmin gpadmin    95 Jun 12 18:28 postmaster.opts
-rw------- 1 gpadmin gpadmin    76 Jun 12 18:28 postmaster.pid
</pre>

#### пременая для главного сегмента
<pre>
1.19)

настраиваем пременную для главного сегмента на мастере
vim /home/gpadmin/.bashrc
export MASTER_DATA_DIRECTORY=/home/gpadmin/data/gpseg-1

пререлогиневаемся и проверяем
exit
su - gpadmin

[gpadmin@client8 ~]$ echo $MASTER_DATA_DIRECTORY
/home/gpadmin/data/gpsne-1
</pre>

#### connect
<pre>
1.20)

подключаемся к gp
[gpadmin@gp1 ~]$ psql -U gpadmin postgres
psql (9.4.26)
Type "help" for help.

postgres=# \l
                               List of databases
   Name    |  Owner  | Encoding |  Collate   |   Ctype    |  Access privileges
-----------+---------+----------+------------+------------+---------------------
 postgres  | gpadmin | UTF8     | en_US.utf8 | en_US.utf8 |
 template0 | gpadmin | UTF8     | en_US.utf8 | en_US.utf8 | =c/gpadmin         +
           |         |          |            |            | gpadmin=CTc/gpadmin
 template1 | gpadmin | UTF8     | en_US.utf8 | en_US.utf8 | =c/gpadmin         +
           |         |          |            |            | gpadmin=CTc/gpadmin
(3 rows)

создал тестовую базу
postgres=# create database test;
CREATE DATABASE

postgres=# \l
                               List of databases
   Name    |  Owner  | Encoding |  Collate   |   Ctype    |  Access privileges
-----------+---------+----------+------------+------------+---------------------
 postgres  | gpadmin | UTF8     | en_US.utf8 | en_US.utf8 |
 template0 | gpadmin | UTF8     | en_US.utf8 | en_US.utf8 | =c/gpadmin         +
           |         |          |            |            | gpadmin=CTc/gpadmin
 template1 | gpadmin | UTF8     | en_US.utf8 | en_US.utf8 | =c/gpadmin         +
           |         |          |            |            | gpadmin=CTc/gpadmin
 test      | gpadmin | UTF8     | en_US.utf8 | en_US.utf8 |
(4 rows)
</pre>

#### смотрим конфигурацию gp
<pre>
1.21)

смотрим конфигурацию gp эта ключевая таблица в которой описываются метаданные нашего кластера
сдесь показаны рабочие узлы  все окей кластер развернули
\c test
test=# select * from gp_segment_configuration;
 dbid | content | role | preferred_role | mode | status | port |         hostname         |         address          |          datadir
------+---------+------+----------------+------+--------+------+--------------------------+--------------------------+----------------------------
    1 |      -1 | p    | p              | n    | u      | 5432 | gp1.ru-central1.internal | gp1.ru-central1.internal | /home/gpadmin/data/gpseg-1
    2 |       0 | p    | p              | n    | u      | 6000 | gp1.ru-central1.internal | gp1.ru-central1.internal | /home/gpadmin/data/gpseg0
    5 |       3 | p    | p              | n    | u      | 6000 | gp2.ru-central1.internal | gp2.ru-central1.internal | /home/gpadmin/data/gpseg3
    3 |       1 | p    | p              | n    | u      | 6001 | gp1.ru-central1.internal | gp1.ru-central1.internal | /home/gpadmin/data/gpseg1
    6 |       4 | p    | p              | n    | u      | 6001 | gp2.ru-central1.internal | gp2.ru-central1.internal | /home/gpadmin/data/gpseg4
    4 |       2 | p    | p              | n    | u      | 6002 | gp1.ru-central1.internal | gp1.ru-central1.internal | /home/gpadmin/data/gpseg2
    7 |       5 | p    | p              | n    | u      | 6002 | gp2.ru-central1.internal | gp2.ru-central1.internal | /home/gpadmin/data/gpseg5
(7 rows)
</pre>

#### тестовые данные
<pre>
1.22)

генерируем числа от одного до миллиона и помещаем в нашу таблицу
create table t1 as select generate_series(1,1000000) as colA distributed by (colA);


проверяем что у нас десять миллионов значений и за какое количество времени выполнился 
test=# EXPLAIN(ANALYZE) select count(*) from t1;
                                                           QUERY PLAN
--------------------------------------------------------------------------------------------------------------------------------
 Aggregate  (cost=0.00..434.43 rows=1 width=8) (actual time=62.505..62.505 rows=1 loops=1)
   ->  Gather Motion 6:1  (slice1; segments: 6)  (cost=0.00..434.43 rows=1 width=8) (actual time=46.483..62.488 rows=6 loops=1)
         ->  Aggregate  (cost=0.00..434.43 rows=1 width=8) (actual time=48.659..48.659 rows=1 loops=1)
               ->  Seq Scan on t1  (cost=0.00..434.12 rows=166667 width=1) (actual time=0.016..34.025 rows=167429 loops=1)
 Planning time: 18.886 ms
   (slice0)    Executor memory: 59K bytes.
   (slice1)    Executor memory: 58K bytes avg x 6 workers, 58K bytes max (seg0).
 Memory used:  128000kB
 Optimizer: Pivotal Optimizer (GPORCA)
 Execution time: 77.551 ms
(10 rows)
</pre>

#### EXPLAIN(ANALYZE) select sum(colA) from t1
<pre>
1.23)

test=# EXPLAIN(ANALYZE) select sum(colA) from t1;
                                                           QUERY PLAN
--------------------------------------------------------------------------------------------------------------------------------
 Aggregate  (cost=0.00..435.43 rows=1 width=8) (actual time=41.686..41.686 rows=1 loops=1)
   ->  Gather Motion 6:1  (slice1; segments: 6)  (cost=0.00..435.43 rows=1 width=8) (actual time=28.170..41.680 rows=6 loops=1)
         ->  Aggregate  (cost=0.00..435.43 rows=1 width=8) (actual time=27.036..27.036 rows=1 loops=1)
               ->  Seq Scan on t1  (cost=0.00..434.12 rows=166667 width=4) (actual time=0.007..15.793 rows=167429 loops=1)
 Planning time: 2.575 ms
   (slice0)    Executor memory: 59K bytes.
   (slice1)    Executor memory: 58K bytes avg x 6 workers, 58K bytes max (seg0).
 Memory used:  128000kB
 Optimizer: Pivotal Optimizer (GPORCA)
 Execution time: 42.343 ms
(10 rows)
</pre>


#### режим распространения зеркала
<pre>
1.24)

Режим распространения зеркала:
(режим распространения, первое зеркало хоста находится на следующем хосте, второе зеркало находится на следующем хосте, а третье зеркало находится на следующем хосте...) 
Выполните команду инициализации: gpinitsystem plus – S, узел Способ распределения раскидистый

gpinitsystem -c gpinitsystem_config -h /home/gpadmin/.hosts -s gp1.ru-central1.internal –S
</pre>

#### если надо удалить кластер gp
<pre>
1.25)

если надо удалить кластер gp
gpdeletesystem

[gpadmin@client8 ~]$ gpdeletesystem

Continue with Greenplum instance deletion? Yy|Nn (default=N):
> y
20230610:22:04:36:016638 gpadmin-[INFO]:-FINAL WARNING, you are about to delete the Greenplum instance
20230610:22:04:36:016638 gpadmin-[INFO]:-on master 

Continue with Greenplum instance deletion? Yy|Nn (default=N):
> y
20230610:22:04:39:016638 gpadmin-[INFO]:-Stopping database...
20230610:22:04:43:016638 gpadmin-[INFO]:-Deleting segments and removing data directories...
20230610:22:04:43:016638 gpadmin-[INFO]:-Waiting for worker threads to complete...
20230610:22:04:43:016638 gpadmin-[INFO]:-Delete system successful.


проверяем данные с каталогами удалены!!!!!
cd /home/gpadmin/data
ls -l
total 0
</pre>



#### 2)

#### Загрузить в неё данные (от 10 до 100 Гб)

<pre>
---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
--------------------------------------------------------------загружаем данные --------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
</pre>


#### данные
<pre>
2.1)

данные
https://console.cloud.google.com/storage/browser/chicago10;tab=objects?prefix=&forceOnObjectsSortingFiltering=false
https://console.cloud.google.com/storage/browser/chicago10;tab=objects?prefix=&forceOnObjectsSortingFiltering=false
https://console.cloud.google.com/storage/browser/chicago10;tab=objects?prefix=&forceOnObjectsSortingFiltering=false
</pre>


#### создать бакет
<pre>
2.2)

создать бакет
https://console.cloud.yandex.ru/folders/b1g4ll61pn5hi287rrrc/storage/create-bucket
https://console.cloud.yandex.ru/folders/b1g4ll61pn5hi287rrrc/storage/buckets

name myotus
name myotus
name myotus
</pre>


#### Загрузил файлы CSV в бакет
<pre>
2.3)

Загрузил файлы CSV, предварительно скаченные с https://console.cloud.google.com/storage/browser/chicago10
</pre>

#### инструкция как пользоваться Как пользоваться S3 API
<pre>
2.4)

инструкция как пользоваться Как пользоваться S3 API
https://cloud.yandex.ru/docs/storage/s3/?from=int-console-empty-state
</pre>

#### создаю сервисный аакаунт
<pre>
2.5)

создаю сервисный аакаунт myvorori
https://console.cloud.yandex.ru/folders/b1g4ll61pn5hi287rrrc?section=service-accounts
https://console.cloud.yandex.ru/folders/b1g4ll61pn5hi287rrrc/storage/buckets
</pre>

#### создаю ключ
<pre>
2.6)

создаю ключ по инструкции
https://cloud.yandex.ru/docs/iam/operations/sa/create-access-key

для доступа сервисного аккаунта создаю сервисный ключ mykey
Идентификатор ключа:
YCAJEGmWpeoKgY6bwncE9rtJs

Ваш секретный ключ:
YCMrCVoA8TpR3dCg8-p4RraHSmuzy0Dp_iClfTBU
Сохраните идентификатор и ключ. После закрытия диалога значение ключа будет недоступно.
</pre>

#### рава доступа к моему бакету
<pre>
2.7)

добавил права доступа к моему бакету myotus для моего сервисного аакаунта myvorori
</pre>

#### s3fs-fuse
<pre>
2.8) 

создаю каталог для данных и устанавливаю s3fs-fuse
yum install s3fs-fuse
mkdir /home/gpadmin/taxi
cd /home/gpadmin/taxi
</pre>

#### .passwd-s3fs
<pre>
2.9)

добавляю ранее созданный индификатор,двоеточие и сервисный ключ
/home/gpadmin/.passwd-s3fs
echo YCAJEGmWpeoKgY6bwncE9rtJs:YCMrCVoA8TpR3dCg8-p4RraHSmuzy0Dp_iClfTBU > /home/gpadmin/.passwd-s3fs
echo YCAJEGmWpeoKgY6bwncE9rtJs:YCMrCVoA8TpR3dCg8-p4RraHSmuzy0Dp_iClfTBU > /home/gpadmin/.passwd-s3fs
echo YCAJEGmWpeoKgY6bwncE9rtJs:YCMrCVoA8TpR3dCg8-p4RraHSmuzy0Dp_iClfTBU > /home/gpadmin/.passwd-s3fs


ПРОВЕРЯЮ!!!!!!!!!
cat /home/gpadmin/.passwd-s3fs
YCAJEGmWpeoKgY6bwncE9rtJs:YCMrCVoA8TpR3dCg8-p4RraHSmuzy0Dp_iClfTBU
</pre>

#### монтирую бакет при помощи s3fs 
<pre>
2.10)

монтирую при помощи s3fs 
s3fs myotus /home/gpadmin/taxi -o passwd_file=/home/gpadmin/.passwd-s3fs -o url=https://storage.yandexcloud.net -o use_path_request_style -o dbglevel=info -f -o curldbg
s3fs myotus /home/gpadmin/taxi -o passwd_file=/home/gpadmin/.passwd-s3fs -o url=https://storage.yandexcloud.net -o use_path_request_style -o dbglevel=info -f -o curldbg
s3fs myotus /home/gpadmin/taxi -o passwd_file=/home/gpadmin/.passwd-s3fs -o url=https://storage.yandexcloud.net -o use_path_request_style -o dbglevel=info -f -o curldbg


----------------------------------------------------------------------------------------
Umask работает как вычитатель, поэтому со всеми нулями он устанавливает значение 777. 
Когда я открываю свое ведро с помощью скрипта, который я сделал, я теперь вижу следующее:
s3fs myotus /home/gpadmin/taxi -o passwd_file=/home/gpadmin/.passwd-s3fs -o url=https://storage.yandexcloud.net -o umask=0000
s3fs myotus /home/gpadmin/taxi -o passwd_file=/home/gpadmin/.passwd-s3fs -o url=https://storage.yandexcloud.net -o umask=0000
s3fs myotus /home/gpadmin/taxi -o passwd_file=/home/gpadmin/.passwd-s3fs -o url=https://storage.yandexcloud.net -o umask=0000
----------------------------------------------------------------------------------------

исходная команда монтирования
s3fs <имя_бакета> $HOME/s3fs -o passwd_file=$HOME/.passwd-s3fs -o url=https://storage.yandexcloud.net -o use_path_request_style -o dbglevel=info -f -o curldbg
s3fs <имя_бакета> $HOME/s3fs -o passwd_file=$HOME/.passwd-s3fs -o url=https://storage.yandexcloud.net -o use_path_request_style -o dbglevel=info -f -o curldbg
s3fs <имя_бакета> $HOME/s3fs -o passwd_file=$HOME/.passwd-s3fs -o url=https://storage.yandexcloud.net -o use_path_request_style -o dbglevel=info -f -o curldbg
</pre>

#### проверяю
<pre>
2.11)
проверяю ура !!!появился новый подключенный раздел s3fs            4.0G     0  4.0G   0% /home/gpadmin/taxi:
[gpadmin@gp1 taxi]$ df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        1.9G     0  1.9G   0% /dev
tmpfs           1.9G   56K  1.9G   1% /dev/shm
tmpfs           1.9G  760K  1.9G   1% /run
tmpfs           1.9G     0  1.9G   0% /sys/fs/cgroup
/dev/vda2        20G  3.0G   18G  15% /
tmpfs           379M     0  379M   0% /run/user/1000
tmpfs           379M     0  379M   0% /run/user/997
s3fs            4.0G     0  4.0G   0% /home/gpadmin/taxi
</pre>

#### проверяю данные
<pre>
2.12) 

видим наши данные в нашем подгруженном бакете
[gpadmin@gp1 ~]$ cd /home/gpadmin/taxi
[gpadmin@gp1 taxi]$ ls -l
total 6782364
-rw-r----- 1 gpadmin gpadmin 267363203 Jun 15 11:52 taxi.csv.000000000000
-rw-r----- 1 gpadmin gpadmin 267344607 Jun 16 01:17 taxi.csv.000000000001
-rw-r----- 1 gpadmin gpadmin 267390243 Jun 16 11:36 taxi.csv.000000000002
-rw-r----- 1 gpadmin gpadmin 267417696 Jun 16 01:17 taxi.csv.000000000003
-rw-r----- 1 gpadmin gpadmin 266371749 Jun 16 11:35 taxi.csv.000000000004
-rw-r----- 1 gpadmin gpadmin 266233035 Jun 17 02:26 taxi.csv.000000000005
-rw-r----- 1 gpadmin gpadmin 266219968 Jun 16 00:35 taxi.csv.000000000006
-rw-r----- 1 gpadmin gpadmin 267380379 Jun 17 02:27 taxi.csv.000000000007
-rw-r----- 1 gpadmin gpadmin 267367985 Jun 16 11:36 taxi.csv.000000000008
-rw-r----- 1 gpadmin gpadmin 267278191 Jun 16 11:36 taxi.csv.000000000009
-rw-r----- 1 gpadmin gpadmin 267389474 Jun 17 02:27 taxi.csv.000000000010
-rw-r----- 1 gpadmin gpadmin 267331103 Jun 17 10:08 taxi.csv.000000000011
-rw-r----- 1 gpadmin gpadmin 266153465 Jun 16 01:17 taxi.csv.000000000012
-rw-r----- 1 gpadmin gpadmin 267368711 Jun 16 01:17 taxi.csv.000000000013
-rw-r----- 1 gpadmin gpadmin 265622273 Jun 16 11:34 taxi.csv.000000000014
-rw-r----- 1 gpadmin gpadmin 267319434 Jun 17 02:27 taxi.csv.000000000015
-rw-r----- 1 gpadmin gpadmin 267311996 Jun 17 02:27 taxi.csv.000000000016
-rw-r----- 1 gpadmin gpadmin 267325169 Jun 17 10:08 taxi.csv.000000000017
-rw-r----- 1 gpadmin gpadmin 267373075 Jun 17 02:27 taxi.csv.000000000018
-rw-r----- 1 gpadmin gpadmin 267377563 Jun 17 10:07 taxi.csv.000000000019
-rw-r----- 1 gpadmin gpadmin 267374748 Jun 17 10:08 taxi.csv.000000000020
-rw-r----- 1 gpadmin gpadmin 267391164 Jun 17 10:08 taxi.csv.000000000021
-rw-r----- 1 gpadmin gpadmin 267335900 Jun 16 11:36 taxi.csv.000000000022
-rw-r----- 1 gpadmin gpadmin 267348008 Jun 16 00:24 taxi.csv.000000000023
-rw-r----- 1 gpadmin gpadmin 267374332 Jun 17 14:08 taxi.csv.000000000024
-rw-r----- 1 gpadmin gpadmin 267369255 Jun 17 13:26 taxi.csv.000000000025
</pre>

#### создаем тестовую бд и таблицу
<pre>
2.13) 

создаем бд и таблицу
Загрузим данные в Постгрес, предварительно создав БД taxi:
psql -U gpadmin postgres -p 5432 -h gp1.ru-central1.internal
psql -U gpadmin postgres -p 5432 -h gp1.ru-central1.internal
psql -U gpadmin postgres -p 5432 -h gp1.ru-central1.internal

CREATE DATABASE taxi;
\c taxi

Чтобы обеспечить равномерное распределение данных, вы хотите выбрать ключ распределения, уникальный для каждой записи, или, если это 
невозможно, выберите РАСПРЕДЕЛЕНИЕ СЛУЧАЙНО. Например:

CREATE TABLE taxi_trips(unique_key text
,taxi_id text
,trip_start_timestamp timestamp
,trip_end_timestamp timestamp
,trip_seconds bigint
,trip_miles float
,pickup_census_tract bigint
,dropoff_census_tract bigint
,pickup_community_area bigint
,dropoff_community_area bigint
,fare float
,tips float
,tolls float
,extras float
,trip_total float
,payment_type text
,company text
,pickup_latitude float
,pickup_longitude float
,pickup_location text
,dropoff_latitude float
,dropoff_longitude float
,dropoff_location text)
DISTRIBUTED RANDOMLY;


create table taxi_trips (
unique_key text, 
taxi_id text, 
trip_start_timestamp TIMESTAMP, 
trip_end_timestamp TIMESTAMP, 
trip_seconds bigint, 
trip_miles numeric, 
pickup_census_tract bigint, 
dropoff_census_tract bigint, 
pickup_community_area bigint, 
dropoff_community_area bigint, 
fare numeric, 
tips numeric, 
tolls numeric, 
extras numeric, 
trip_total numeric, 
payment_type text, 
company text, 
pickup_latitude numeric, 
pickup_longitude numeric, 
pickup_location text, 
dropoff_latitude numeric, 
dropoff_longitude numeric, 
dropoff_location text
);
</pre>

#### команда на загрузку данных в цикле 
<pre>
2.14) 

команда на загрузку данных в цикле 
for f in *.csv*; do psql -U gpadmin -p 5432 -h gp1.ru-central1.internal -d taxi -c "\\COPY taxi_trips FROM PROGRAM 'cat $f' CSV HEADER"; done
for f in *.csv*; do psql -U gpadmin -p 5432 -h gp1.ru-central1.internal -d taxi -c "\\COPY taxi_trips FROM PROGRAM 'cat $f' CSV HEADER"; done
for f in *.csv*; do psql -U gpadmin -p 5432 -h gp1.ru-central1.internal -d taxi -c "\\COPY taxi_trips FROM PROGRAM 'cat $f' CSV HEADER"; done
COPY 668818
COPY 669331
COPY 668352
COPY 669666
COPY 667789
COPY 628731
COPY 635790
COPY 669381
COPY 670047
COPY 672486
COPY 669053
COPY 681872
COPY 662644
COPY 669215
COPY 919506
COPY 671838
COPY 671997
COPY 672941
COPY 669019
COPY 669107
COPY 668928
COPY 669285
COPY 670989
COPY 671264
COPY 669396
COPY 670978
</pre>

#### выполняем заливку при пормощи  gpfdist
<pre>
2.15) 

выполняем заливку при пормощи  gpfdist
https://docs.vmware.com/en/VMware-Greenplum/7/greenplum-database/utility_guide-ref-gpfdist.html

psql -U gpadmin -p 5432 -h gp1.ru-central1.internal -d taxi
select * from taxi_trips2;

заускаем gpfdist
gpfdist -d /home/gpadmin/taxi -p 8081 > gpfdist.log 2>&1 &
gpfdist -d /home/gpadmin/taxi -p 8081 > gpfdist.log 2>&1 &
gpfdist -d /home/gpadmin/taxi -p 8081 > gpfdist.log 2>&1 &
</pre>

#### если надо остановить gpfdist
<pre>
2.16)

если надо остановить gpfdist
Чтобы остановить gpfdist, когда он работает в фоновом режиме:
Сначала найдите его идентификатор процесса:
$ ps -ef | grep gpfdist
Затем остановите процесс, например (где 3456 — это идентификатор процесса в этом примере):
$ kill 3456
</pre>

#### status gpfdist
<pre>
2.17)

проверяем лог что gpfdist работает и все ок 
[gpadmin@gp1 ~]$ cat gpfdist.log
2023-06-19 15:34:27 1984 INFO Before opening listening sockets - following listening sockets are available:
2023-06-19 15:34:27 1984 INFO IPV6 socket: [::]:8081
2023-06-19 15:34:27 1984 INFO IPV4 socket: 0.0.0.0:8081
2023-06-19 15:34:27 1984 INFO Trying to open listening socket:
2023-06-19 15:34:27 1984 INFO IPV6 socket: [::]:8081
2023-06-19 15:34:27 1984 INFO Opening listening socket succeeded
2023-06-19 15:34:27 1984 INFO Trying to open listening socket:
2023-06-19 15:34:27 1984 INFO IPV4 socket: 0.0.0.0:8081
2023-06-19 15:34:27 1984 INFO Opening listening socket succeeded
Serving HTTP on port 8081, directory /home/gpadmin/taxi
</pre>

#### создаем внешнюю/внешнюю веб-таблицу и заливаем из нее данные в нашу
<pre>
2.18)

создаем внешнюю/внешнюю веб-таблицу и заливаем из нее данные в нашу
CREATE READABLE EXTERNAL TABLE taxi_trips_ext (like taxi_trips2)
LOCATION('gpfdist://127.0.0.1:8081/taxi.csv.000000000000')
FORMAT 'csv' (header)
LOG ERRORS SEGMENT REJECT LIMIT 50 rows;

INSERT INTO taxi_trips2  SELECT * FROM taxi_trips_ext;
INSERT 0 668818
</pre>



#### 3)

#### Сравнить скорость выполнения запросов на PosgreSQL и выбранной СУБД

<pre>
---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------кластер на postgres 15 -------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
</pre>

#### создал и установил с подтюнил postgres 15
<pre>
3.1) 

создал и установил с подтюнил postgres 15 озу 6 gb CPU 2 диск hdd
влил тот же объем данных что и на кластер gp

ALTER SYSTEM SET
 max_connections = '40';
ALTER SYSTEM SET
 shared_buffers = '1536MB';
ALTER SYSTEM SET
 effective_cache_size = '4608MB';
ALTER SYSTEM SET
 maintenance_work_mem = '500MB';
ALTER SYSTEM SET
 checkpoint_completion_target = '0.9';
ALTER SYSTEM SET
 wal_buffers = '16MB';
ALTER SYSTEM SET
 default_statistics_target = '100';
ALTER SYSTEM SET
 random_page_cost = '4';
ALTER SYSTEM SET
 effective_io_concurrency = '2';
ALTER SYSTEM SET
 work_mem = '19660kB';
ALTER SYSTEM SET
 min_wal_size = '1GB';
ALTER SYSTEM SET
 max_wal_size = '4GB';

sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
sudo yum -y repolist
sudo yum install postgresql15-contrib

sudo /usr/pgsql-15/bin/postgresql-15-setup initdb
systemctl enable postgresql-15.service
systemctl status postgresql-15.service
systemctl restart postgresql-15.service

create user gpadmin;
create database taxi with owner gpadmin;
s3fs myotus /home/gpadmin/ttt -o passwd_file=/home/gpadmin/.passwd-s3fs -o url=https://storage.yandexcloud.net -o umask=0000
for f in *.csv*; do psql -U gpadmin  -d taxi -c "\\COPY taxi_trips FROM PROGRAM 'cat $f' CSV HEADER"; done
</pre>


<pre>
---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------итого по запросам ------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
</pre>


#### прогнал запросы big:
<pre>
3.2)

---------------------------------------------------------------------------------------------------------------------------------------
запрос 
EXPLAIN(ANALYZE) SELECT payment_type, round(sum(tips)/sum(trip_total)*100, 0) + 0 as tips_percent, count(*) as c 
FROM taxi_trips
group by payment_type
order by c;

на gp 2 ноды  6 сегментов
Execution time: 3629.583 ms

на ванильном Postgre
Execution Time: 107498.652 ms
---------------------------------------------------------------------------------------------------------------------------------------
</pre>


#### тестовая генерация данных
<pre>
3.3)

---------------------------------------------------------------------------------------------------------------------------------------
тестовая генерация данных:
create table t1 as select generate_series(1,1000000) as colA distributed by (colA);

на gp 2 ноды  6 сегментов
Execution time: 1.583 ms

на ванильном Postgre
Execution time: 2.884 ms
---------------------------------------------------------------------------------------------------------------------------------------
</pre>

#### select count(*) from t1;
<pre>
3.4)

---------------------------------------------------------------------------------------------------------------------------------------
запрос:
EXPLAIN(ANALYZE) select count(*) from t1;

на gp 2 ноды  6 сегментов
Execution time: 92.168 ms

на ванильном Postgre
Execution Time: 84.369 ms
---------------------------------------------------------------------------------------------------------------------------------------
</pre>

#### EXPLAIN(ANALYZE) select sum(colA) from t1;
<pre>
3.5)

---------------------------------------------------------------------------------------------------------------------------------------
запрос:
EXPLAIN(ANALYZE) select sum(colA) from t1;

на gp 2 ноды  6 сегментов
Execution time: 88.765 ms

на ванильном Postgre
Execution Time: 102.372 ms
---------------------------------------------------------------------------------------------------------------------------------------
</pre>

#### 4)

#### Описать что и как делали и с какими проблемами столкнулись

<pre>
---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------трудности---------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
</pre>

<pre>
первый раз работал с gp разбирал ошибки когда кластер не поднимался
изучал документацию gp чтобы корректно поднять кластер на двух ВМ и шести сегментов
с самого начала поднял gp на одной VM но мне показалось это не очень круто

были шероховатиости  в работе с бакетом и s3fs 
потратил достаточно большое количество времени чтобы устранить ошибки подключения и доступа
</pre>