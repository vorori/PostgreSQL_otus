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
----------------------------------мануал создания мастер + три ноды  = 12 segment START--------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
</pre>


#### подготовительные мероприятия 
<pre>
1.1) 
 
gpmaster  10.129.0.26      ssh vorori@51.250.111.41
gp1       10.129.0.15      ssh vorori@158.160.71.97
gp2       10.129.0.8       ssh vorori@84.252.139.146
gp3       10.129.0.25      ssh vorori@158.160.21.11

sudo yum -y install epel-release && yum -y install htop mc vim wget telnet
sudo yum -y install epel-release && yum -y install htop mc vim wget telnet
sudo yum -y install epel-release && yum -y install htop mc vim wget telnet

hostname && hostname -i


cat /etc/hosts 

cat >> /etc/hosts <<EOF
# Greenplum DB
10.129.0.26   gpmaster.ru-central1.internal
10.129.0.15   gp1.ru-central1.internal
10.129.0.8    gp2.ru-central1.internal
10.129.0.25   gp3.ru-central1.internal
EOF

#отключить SELinux status: disabled
vim /etc/selinux/config
cat /etc/selinux/config
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=disabled/' /etc/selinux/config
reboot -h now

tail -f /var/log/messages
tail -f /var/log/messages
tail -f /var/log/messages


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

cat /etc/sysctl.conf

sudo cat <<EOF>> /etc/sysctl.conf
vm.overcommit_memory = 2
vm.overcommit_ratio = 95 
net.ipv4.ip_local_port_range = 10000 65535 
kernel.sem = 250 2048000 200 8192
EOF

sysctl --system


#### создать столько файлов и процессов вплодь до этого числа
<pre>
1.4)

cat /etc/security/limits.conf 

sudo cat <<EOF>> /etc/security/limits.conf 
* soft nofile 524288
* hard nofile 524288
* soft nproc 131072
* hard nproc 131072
EOF
</pre>


#### добавляем групу gpadmin
<pre>
1.5)

groupadd gpadmin
useradd gpadmin -r -m -g gpadmin
echo my_pass | passwd --stdin gpadmin
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

cd /tmp
wget https://github.com/greenplum-db/gpdb/releases/download/6.24.3/open-source-greenplum-db-6.24.3-rhel7-x86_64.rpm
yum install ./open-source-greenplum-db-6.24.3-rhel7-x86_64.rpm


============================================================================================================================== Package                         Arch        Version                 Repository                                          Size
==============================================================================================================================Installing:
 open-source-greenplum-db-6      x86_64      6.24.3-1.el7            /open-source-greenplum-db-6.24.3-rhel7-x86_64      295 M
Installing for dependencies:
 apr                             x86_64      1.4.8-7.el7             base                                               104 k
 apr-util                        x86_64      1.5.2-6.el7_9.1         updates                                             92 k
 bzip2                           x86_64      1.0.6-13.el7            base                                                52 k
 keyutils-libs-devel             x86_64      1.5.8-3.el7             base                                                37 k
 krb5-devel                      x86_64      1.15.1-55.el7_9         updates                                            273 k
 libcgroup-tools                 x86_64      0.41-21.el7             base                                                99 k
 libcom_err-devel                x86_64      1.42.9-19.el7           base                                                32 k
 libkadm5                        x86_64      1.15.1-55.el7_9         updates                                            180 k
 libselinux-devel                x86_64      2.5-15.el7              base                                               187 k
 libsepol-devel                  x86_64      2.5-10.el7              base                                                77 k
 libverto-devel                  x86_64      0.2.5-4.el7             base                                                12 k
 pcre-devel                      x86_64      8.32-17.el7             base                                               480 k
 zip                             x86_64      3.0-11.el7              base                                               260 k

Transaction Summary
==============================================================================================================================Install  1 Package (+13 Dependent packages)

Total size: 297 M
Total download size: 1.8 M
Installed size: 299 M
Is this ok [y/d/N]: y
</pre>

#### проверяем
<pre>
1.8)

ls -l /usr/local


drwxr-xr-x.  2 root root   6 Apr 11  2018 bin
drwxr-xr-x.  2 root root   6 Apr 11  2018 etc
drwxr-xr-x.  2 root root   6 Apr 11  2018 games
lrwxrwxrwx   1 root root  30 Jul 10 13:35 greenplum-db -> /usr/local/greenplum-db-6.24.3
drwxr-xr-x  11 root root 238 Jul 10 13:35 greenplum-db-6.24.3
drwxr-xr-x.  2 root root   6 Apr 11  2018 include
drwxr-xr-x.  2 root root   6 Apr 11  2018 lib
drwxr-xr-x.  2 root root   6 Apr 11  2018 lib64
drwxr-xr-x.  2 root root   6 Apr 11  2018 libexec
drwxr-xr-x.  2 root root   6 Apr 11  2018 sbin
drwxr-xr-x.  5 root root  49 Feb  2  2021 share
drwxr-xr-x.  2 root root   6 Apr 11  2018 src
</pre>

####  заходим под пользователем gpadmin для установки настройки
<pre>
1.9)

su - gpadmin

#настравиваем среду к файлу greenplum_path.sh
vim /home/gpadmin/.bashrc
----------------------------------------------------
# User specific aliases and functions
. /usr/local/greenplum-db/greenplum_path.sh
----------------------------------------------------

#презайти
exit
logout
su - gpadmin

cd $GPHOME/
which psql
/usr/local/greenplum-db-6.24.3/bin/psql
</pre>

#### Создайте файл с именем hosts в домашнем каталоге пользователя gpadmin, который содержит все имена хостов Greenplum
<pre>
1.10)

vim /home/gpadmin/.hosts
vim /home/gpadmin/.hosts
vim /home/gpadmin/.hosts
-------------------------------
gpmaster.ru-central1.internal
gp1.ru-central1.internal
gp2.ru-central1.internal
gp3.ru-central1.internal
--------------------------------
</pre>



1.11.1)

выполняем следующие шаги на главном хосте в качестве пользователя gpadmin.
генерирую ключ (на второй ноде тоже чтобы создать каталог ./ssh)
su - gpadmin
[gpadmin@gpmaster ~]$ ssh-keygen -t rsa
                      ssh-keygen -t rsa -b 4096
					  ssh-keygen -t rsa -b 4096
					  ssh-keygen -t rsa -b 4096
                      
Generating public/private rsa key pair.
Enter file in which to save the key (/home/gpadmin/.ssh/id_rsa):
Created directory '/home/gpadmin/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/gpadmin/.ssh/id_rsa.
Your public key has been saved in /home/gpadmin/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:A2pFF+Kq1nDoKJXKSbJdBocCmeoYl70Enm2TOYc6UG0 gpadmin@gpmaster.ru-central1.internal
The key's randomart image is:
+---[RSA 4096]----+
|.o .  o o.       |
|+ o.Eo o         |
|oooB.++          |
|+.==@+..         |
|+==+B=  S        |
|=O+O.    .       |
|=o=..            |
|..               |
|                 |
+----[SHA256]-----+
</pre>

#### передаю публичный ключ на сервер ноду 2
1.12.2)
<pre>
#после генерации мне надо предать этот публичный ключ на все сервера
ssh-copy-id -i /home/gpadmin/.ssh/id_rsa.pub gpadmin@gp1.ru-central1.internal -p 22
ssh-copy-id -i /home/gpadmin/.ssh/id_rsa.pub gpadmin@gp2.ru-central1.internal -p 22
ssh-copy-id -i /home/gpadmin/.ssh/id_rsa.pub gpadmin@gp3.ru-central1.internal -p 22


#проверяем что можем подключиться
ssh gpadmin@gp1.ru-central1.internal
ssh gpadmin@gp2.ru-central1.internal
ssh gpadmin@gp3.ru-central1.internal
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

#проверяем что можем подключиться
ssh gpadmin@gp1.ru-central1.internal
ssh gpadmin@gp2.ru-central1.internal
ssh gpadmin@gp3.ru-central1.internal
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

#проверяем что можем подключиться
ssh gpadmin@gp1.ru-central1.internal
ssh gpadmin@gp2.ru-central1.internal
ssh gpadmin@gp3.ru-central1.internal
-----------------------------------------------------------------------------------------------------------------------------------------------------


выполняем на главном gpmaster.ru-central1.internal
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
  ... send to gp1.ru-central1.internal
  ... send to gp2.ru-central1.internal
  ... send to gp3.ru-central1.internal

[STEP 4 of 5] determine common authentication file content

[STEP 5 of 5] copy authentication files to all remote hosts
  ... finished key exchange with gp1.ru-central1.internal
  ... finished key exchange with gp2.ru-central1.internal
  ... finished key exchange with gp3.ru-central1.internal

[INFO] completed successfully

</pre>
	 
#### проверяю ВСЕ КЛЮЧИ
<pre>
1.13.1)

на сервере ноде ДОЛДЖНЫ ПОЯВИТЬСЯ ВСЕ КЛЮЧИ!
появятся вот эти файлы причем и на ноде которую мы подключили
ls -l /home/gpadmin/.ssh
ls -l /home/gpadmin/.ssh

[gpadmin@gp2 greenplum-db-6.24.3]$ ls -l /home/gpadmin/.ssh
total 16
-rw------- 1 gpadmin gpadmin  763 Jul 10 14:36 authorized_keys
-rw-r--r-- 1 gpadmin gpadmin    0 Jul 10 14:36 config
-rw-r--r-- 1 gpadmin gpadmin    0 Jul 10 14:36 iddummy.pub
-rw------- 1 gpadmin gpadmin 3243 Jul 10 14:36 id_rsa
-rw-r--r-- 1 gpadmin gpadmin  763 Jul 10 14:36 id_rsa.pub
-rw-rw-r-- 1 gpadmin gpadmin 2615 Jul 10 14:36 known_hosts
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

------------------------------------------------------------------
[gpadmin@gpmaster .ssh]$ gpssh -f /home/gpadmin/.hosts
=> mkdir /home/gpadmin/data
[     gp3.ru-central1.internal]
[     gp2.ru-central1.internal]
[     gp1.ru-central1.internal]
[gpmaster.ru-central1.internal]
=> mkdir /home/gpadmin/data_mirror
[     gp3.ru-central1.internal]
[     gp2.ru-central1.internal]
[     gp1.ru-central1.internal]
[gpmaster.ru-central1.internal]
------------------------------------------------------------------

проверяю созданые каталоги на серверах
ls -l /home/gpadmin
total 0
drwxrwxr-x 2 gpadmin gpadmin 6 Jul 10 14:41 data
drwxrwxr-x 2 gpadmin gpadmin 6 Jul 10 14:42 data_mirror
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
MASTER_HOSTNAME=gpmaster.ru-central1.internal

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

gpinitsystem -c /home/gpadmin/gpinitsystem_config -s gpmaster.ru-central1.internal
gpinitsystem -c /home/gpadmin/gpinitsystem_config -s gpmaster.ru-central1.internal
gpinitsystem -c /home/gpadmin/gpinitsystem_config -s gpmaster.ru-central1.internal

[gpadmin@gpmaster gpconfigs]$ gpinitsystem -c /home/gpadmin/gpinitsystem_config -s gpmaster.ru-central1.internal
20230710:18:45:34:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Checking configuration parameters, please wait...
20230710:18:45:34:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Reading Greenplum configuration file /home/gpadmin/gpinitsystem_config
20230710:18:45:34:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Locale has not been set in /home/gpadmin/gpinitsystem_config, will set to default value
20230710:18:45:34:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Locale set to en_US.utf8
20230710:18:45:34:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-No DATABASE_NAME set, will exit following template1 updates
20230710:18:45:34:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-MASTER_MAX_CONNECT not set, will set to default value 250
20230710:18:45:35:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Checking configuration parameters, Completed
20230710:18:45:35:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Commencing multi-home checks, please wait...
....
20230710:18:45:37:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Configuring build for standard array
20230710:18:45:37:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Commencing multi-home checks, Completed
20230710:18:45:37:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Building primary segment instance array, please wait...
............
20230710:18:45:41:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Checking Master host
20230710:18:45:41:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Checking new segment hosts, please wait...
............
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Checking new segment hosts, Completed
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Greenplum Database Creation Parameters
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:---------------------------------------
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Master Configuration
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:---------------------------------------
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Master instance name       = Greenplum Data Platform
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Master hostname            = gpmaster.ru-central1.internal
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Master port                = 5432
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Master instance dir        = /home/gpadmin/data/gpseg-1
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Master LOCALE              = en_US.utf8
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Greenplum segment prefix   = gpseg
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Master Database            =
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Master connections         = 250
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Master buffers             = 128000kB
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Segment connections        = 750
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Segment buffers            = 128000kB
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Checkpoint segments        = 8
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Encoding                   = UNICODE
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Postgres param file        = Off
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Initdb to be used          = /usr/local/greenplum-db-6.24.3/bin/initdb
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-GP_LIBRARY_PATH is         = /usr/local/greenplum-db-6.24.3/lib
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-HEAP_CHECKSUM is           = on
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-HBA_HOSTNAMES is           = 0
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Ulimit check               = Passed
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Array host connect type    = Single hostname per node
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Master IP address [1]      = ::1
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Master IP address [2]      = 10.129.0.26
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Master IP address [3]      = fe80::d20d:14ff:fea3:b5ce
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Standby Master             = gpmaster.ru-central1.internal
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Number of primary segments = 3
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Standby IP address         = ::1
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Standby IP address         = 10.129.0.26
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Standby IP address         = fe80::d20d:14ff:fea3:b5ce
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Total Database segments    = 12
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Trusted shell              = ssh
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Number segment hosts       = 4
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Mirroring config           = OFF
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:----------------------------------------
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Greenplum Primary Segment Configuration
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:----------------------------------------
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-gpmaster.ru-central1.internal    6000    gpmaster.ru-central1.internal                                                                                                       /home/gpadmin/data/gpseg0        2
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-gpmaster.ru-central1.internal    6001    gpmaster.ru-central1.internal                                                                                                       /home/gpadmin/data/gpseg1        3
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-gpmaster.ru-central1.internal    6002    gpmaster.ru-central1.internal                                                                                                       /home/gpadmin/data/gpseg2        4
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-gp1.ru-central1.internal         6000    gp1.ru-central1.internal                                                                                                            /home/gpadmin/data/gpseg3        5
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-gp1.ru-central1.internal         6001    gp1.ru-central1.internal                                                                                                            /home/gpadmin/data/gpseg4        6
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-gp1.ru-central1.internal         6002    gp1.ru-central1.internal                                                                                                            /home/gpadmin/data/gpseg5        7
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-gp2.ru-central1.internal         6000    gp2.ru-central1.internal                                                                                                            /home/gpadmin/data/gpseg6        8
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-gp2.ru-central1.internal         6001    gp2.ru-central1.internal                                                                                                            /home/gpadmin/data/gpseg7        9
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-gp2.ru-central1.internal         6002    gp2.ru-central1.internal                                                                                                            /home/gpadmin/data/gpseg8        10
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-gp3.ru-central1.internal         6000    gp3.ru-central1.internal                                                                                                            /home/gpadmin/data/gpseg9        11
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-gp3.ru-central1.internal         6001    gp3.ru-central1.internal                                                                                                            /home/gpadmin/data/gpseg10       12
20230710:18:45:51:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-gp3.ru-central1.internal         6002    gp3.ru-central1.internal                                                                                                            /home/gpadmin/data/gpseg11       13

Continue with Greenplum creation Yy|Nn (default=N):
> y
20230710:18:46:30:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Building the Master instance database, please wait...
20230710:18:46:37:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Starting the Master in admin mode
20230710:18:46:39:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Commencing parallel build of primary segment instances
20230710:18:46:39:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Spawning parallel processes    batch [1], please wait...
............
20230710:18:46:39:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Waiting for parallel processes batch [1], please wait...
.......................
20230710:18:47:02:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:------------------------------------------------
20230710:18:47:02:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Parallel process exit status
20230710:18:47:02:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:------------------------------------------------
20230710:18:47:02:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Total processes marked as completed           = 12
20230710:18:47:02:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Total processes marked as killed              = 0
20230710:18:47:02:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Total processes marked as failed              = 0
20230710:18:47:02:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:------------------------------------------------
20230710:18:47:02:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Removing back out file
20230710:18:47:02:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-No errors generated from parallel processes
20230710:18:47:02:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Restarting the Greenplum instance in production mode
20230710:18:47:07:011770 gpstop:gpmaster:gpadmin-[INFO]:-Starting gpstop with args: -a -l /home/gpadmin/gpAdminLogs -m -d /home/gpadmin/data/gpseg-1
20230710:18:47:07:011770 gpstop:gpmaster:gpadmin-[INFO]:-Gathering information and validating the environment...
20230710:18:47:07:011770 gpstop:gpmaster:gpadmin-[INFO]:-Obtaining Greenplum Master catalog information
20230710:18:47:07:011770 gpstop:gpmaster:gpadmin-[INFO]:-Obtaining Segment details from master...
20230710:18:47:07:011770 gpstop:gpmaster:gpadmin-[INFO]:-Greenplum Version: 'postgres (Greenplum Database) 6.24.3 build commit:25d3498a400ca5230e81abb94861f23389315213 Open Source'
20230710:18:47:07:011770 gpstop:gpmaster:gpadmin-[INFO]:-Commencing Master instance shutdown with mode='smart'
20230710:18:47:07:011770 gpstop:gpmaster:gpadmin-[INFO]:-Master segment instance directory=/home/gpadmin/data/gpseg-1
20230710:18:47:07:011770 gpstop:gpmaster:gpadmin-[INFO]:-Stopping master segment and waiting for user connections to finish ...
server shutting down
20230710:18:47:08:011770 gpstop:gpmaster:gpadmin-[INFO]:-Attempting forceful termination of any leftover master process
20230710:18:47:08:011770 gpstop:gpmaster:gpadmin-[INFO]:-Terminating processes for segment /home/gpadmin/data/gpseg-1
20230710:18:47:09:012004 gpstart:gpmaster:gpadmin-[INFO]:-Starting gpstart with args: -a -l /home/gpadmin/gpAdminLogs -d /home/gpadmin/data/gpseg-1
20230710:18:47:09:012004 gpstart:gpmaster:gpadmin-[INFO]:-Gathering information and validating the environment...
20230710:18:47:09:012004 gpstart:gpmaster:gpadmin-[INFO]:-Greenplum Binary Version: 'postgres (Greenplum Database) 6.24.3 build commit:25d3498a400ca5230e81abb94861f23389315213 Open Source'
20230710:18:47:09:012004 gpstart:gpmaster:gpadmin-[INFO]:-Greenplum Catalog Version: '301908232'
20230710:18:47:09:012004 gpstart:gpmaster:gpadmin-[INFO]:-Starting Master instance in admin mode
20230710:18:47:09:012004 gpstart:gpmaster:gpadmin-[INFO]:-Obtaining Greenplum Master catalog information
20230710:18:47:09:012004 gpstart:gpmaster:gpadmin-[INFO]:-Obtaining Segment details from master...
20230710:18:47:09:012004 gpstart:gpmaster:gpadmin-[INFO]:-Setting new master era
20230710:18:47:09:012004 gpstart:gpmaster:gpadmin-[INFO]:-Master Started...
20230710:18:47:10:012004 gpstart:gpmaster:gpadmin-[INFO]:-Shutting down master
20230710:18:47:11:012004 gpstart:gpmaster:gpadmin-[INFO]:-Commencing parallel segment instance startup, please wait...
........
20230710:18:47:19:012004 gpstart:gpmaster:gpadmin-[INFO]:-Process results...
20230710:18:47:19:012004 gpstart:gpmaster:gpadmin-[INFO]:-----------------------------------------------------
20230710:18:47:19:012004 gpstart:gpmaster:gpadmin-[INFO]:-   Successful segment starts                                            = 12
20230710:18:47:19:012004 gpstart:gpmaster:gpadmin-[INFO]:-   Failed segment starts                                                = 0
20230710:18:47:19:012004 gpstart:gpmaster:gpadmin-[INFO]:-   Skipped segment starts (segments are marked down in configuration)   = 0
20230710:18:47:19:012004 gpstart:gpmaster:gpadmin-[INFO]:-----------------------------------------------------
20230710:18:47:19:012004 gpstart:gpmaster:gpadmin-[INFO]:-Successfully started 12 of 12 segment instances
20230710:18:47:19:012004 gpstart:gpmaster:gpadmin-[INFO]:-----------------------------------------------------
20230710:18:47:19:012004 gpstart:gpmaster:gpadmin-[INFO]:-Starting Master instance gpmaster.ru-central1.internal directory /home/gpadmin/data/gpseg-1
20230710:18:47:20:012004 gpstart:gpmaster:gpadmin-[INFO]:-Command pg_ctl reports Master gpmaster.ru-central1.internal instance active
20230710:18:47:20:012004 gpstart:gpmaster:gpadmin-[INFO]:-Connecting to dbname='template1' connect_timeout=15
20230710:18:47:20:012004 gpstart:gpmaster:gpadmin-[INFO]:-No standby master configured.  skipping...
20230710:18:47:20:012004 gpstart:gpmaster:gpadmin-[INFO]:-Database successfully started
20230710:18:47:20:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Completed restart of Greenplum instance in production mode
20230710:18:47:20:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Starting initialization of standby master gpmaster.ru-central1.internal
20230710:18:47:20:012418 gpinitstandby:gpmaster:gpadmin-[INFO]:-Validating environment and parameters for standby initialization...
20230710:18:47:21:012418 gpinitstandby:gpmaster:gpadmin-[INFO]:-Checking for data directory /home/gpadmin/data/gpseg-1 on gpmaster.ru-central1.internal
20230710:18:47:21:001375 gpinitsystem:gpmaster:gpadmin-[WARN]:-Failed to complete standby master initialization
20230710:18:47:21:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Scanning utility log file for any warning messages
20230710:18:47:21:001375 gpinitsystem:gpmaster:gpadmin-[WARN]:-*******************************************************
20230710:18:47:21:001375 gpinitsystem:gpmaster:gpadmin-[WARN]:-Scan of log file indicates that some warnings or errors
20230710:18:47:21:001375 gpinitsystem:gpmaster:gpadmin-[WARN]:-were generated during the array creation
20230710:18:47:21:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Please review contents of log file
20230710:18:47:21:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-/home/gpadmin/gpAdminLogs/gpinitsystem_20230710.log
20230710:18:47:21:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-To determine level of criticality
20230710:18:47:21:001375 gpinitsystem:gpmaster:gpadmin-[WARN]:-*******************************************************
20230710:18:47:21:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Greenplum Database instance successfully created
20230710:18:47:21:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-------------------------------------------------------
20230710:18:47:21:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-To complete the environment configuration, please
20230710:18:47:21:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-update gpadmin .bashrc file with the following
20230710:18:47:21:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-1. Ensure that the greenplum_path.sh file is sourced
20230710:18:47:21:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-2. Add "export MASTER_DATA_DIRECTORY=/home/gpadmin/data/gpseg-1"
20230710:18:47:21:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-   to access the Greenplum scripts for this instance:
20230710:18:47:21:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-   or, use -d /home/gpadmin/data/gpseg-1 option for the Greenplum scripts
20230710:18:47:21:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-   Example gpstate -d /home/gpadmin/data/gpseg-1
20230710:18:47:21:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Script log file = /home/gpadmin/gpAdminLogs/gpinitsystem_20230710.log
20230710:18:47:21:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-To remove instance, run gpdeletesystem utility
20230710:18:47:21:001375 gpinitsystem:gpmaster:gpadmin-[WARN]:-Standby Master failed to initialize
20230710:18:47:21:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-------------------------------------------------------
20230710:18:47:21:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-The Master /home/gpadmin/data/gpseg-1/pg_hba.conf post gpinitsystem
20230710:18:47:21:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-has been configured to allow all hosts within this new
20230710:18:47:21:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-array to intercommunicate. Any hosts external to this
20230710:18:47:21:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-new array must be explicitly added to this file
20230710:18:47:21:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Refer to the Greenplum Admin support guide which is
20230710:18:47:21:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-located in the /usr/local/greenplum-db-6.24.3/docs directory
20230710:18:47:21:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-------------------------------------------------------
20230710:18:47:21:001375 gpinitsystem:gpmaster:gpadmin-[WARN]:-*******************************************************
20230710:18:47:21:001375 gpinitsystem:gpmaster:gpadmin-[WARN]:-Cluster setup finished, but Standby Master failed to initialize. Review contents of log files for errors.
20230710:18:47:21:001375 gpinitsystem:gpmaster:gpadmin-[INFO]:-Use gpinitstandby to create a Standby Master
20230710:18:47:21:001375 gpinitsystem:gpmaster:gpadmin-[WARN]:-*******************************************************
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
[gpadmin@gpmaster gpconfigs]$ ls -l /home/gpadmin/data
total 16
drwx------ 21 gpadmin gpadmin 4096 Jul 10 18:47 gpseg0
drwx------ 21 gpadmin gpadmin 4096 Jul 10 18:47 gpseg1
drwx------ 22 gpadmin gpadmin 4096 Jul 10 18:47 gpseg-1
drwx------ 21 gpadmin gpadmin 4096 Jul 10 18:47 gpseg2


проверка сегментов проверяем на всех серверах
[gpadmin@gp1 ~]$ ls -l /home/gpadmin/data
total 12
drwx------ 21 gpadmin gpadmin 4096 Jul 10 18:47 gpseg3
drwx------ 21 gpadmin gpadmin 4096 Jul 10 18:47 gpseg4
drwx------ 21 gpadmin gpadmin 4096 Jul 10 18:47 gpseg5


[gpadmin@gp2 ~]$ ls -l /home/gpadmin/data
total 12
drwx------ 21 gpadmin gpadmin 4096 Jul 10 18:47 gpseg6
drwx------ 21 gpadmin gpadmin 4096 Jul 10 18:47 gpseg7
drwx------ 21 gpadmin gpadmin 4096 Jul 10 18:47 gpseg8


[gpadmin@gp3 ~]$ ls -l /home/gpadmin/data
total 12
drwx------ 21 gpadmin gpadmin 4096 Jul 10 18:47 gpseg10
drwx------ 21 gpadmin gpadmin 4096 Jul 10 18:47 gpseg11
drwx------ 21 gpadmin gpadmin 4096 Jul 10 18:47 gpseg9

</pre>

#### описание главный сегмент
<pre>
1.18)

главный сегмент в названии будет отрицательное число в нашем случае -1
ls -l /home/gpadmin/data/gpseg-1
[gpadmin@gpmaster gpconfigs]$ ls -l /home/gpadmin/data/gpseg-1
total 72
drwx------ 5 gpadmin gpadmin    41 Jul 10 18:46 base
drwx------ 2 gpadmin gpadmin  4096 Jul 10 18:47 global
drwxrwxr-x 5 gpadmin gpadmin    42 Jul 10 18:46 gpperfmon
-rw------- 1 gpadmin gpadmin   918 Jul 10 18:47 gpsegconfig_dump
-rw-rw-r-- 1 gpadmin gpadmin   860 Jul 10 18:46 gpssh.conf
-rw------- 1 gpadmin gpadmin    10 Jul 10 18:46 internal.auto.conf
drwx------ 2 gpadmin gpadmin    18 Jul 10 18:46 pg_clog
drwx------ 2 gpadmin gpadmin    18 Jul 10 18:46 pg_distributedlog
drwx------ 2 gpadmin gpadmin     6 Jul 10 18:46 pg_dynshmem
-rw-rw-r-- 1 gpadmin gpadmin  4818 Jul 10 18:46 pg_hba.conf
-rw------- 1 gpadmin gpadmin  1636 Jul 10 18:46 pg_ident.conf
drwx------ 2 gpadmin gpadmin   141 Jul 10 18:47 pg_log
drwx------ 4 gpadmin gpadmin    39 Jul 10 18:46 pg_logical
drwx------ 4 gpadmin gpadmin    36 Jul 10 18:46 pg_multixact
drwx------ 2 gpadmin gpadmin    18 Jul 10 18:47 pg_notify
drwx------ 2 gpadmin gpadmin     6 Jul 10 18:46 pg_replslot
drwx------ 2 gpadmin gpadmin     6 Jul 10 18:46 pg_serial
drwx------ 2 gpadmin gpadmin     6 Jul 10 18:46 pg_snapshots
drwx------ 2 gpadmin gpadmin     6 Jul 10 18:47 pg_stat
drwx------ 2 gpadmin gpadmin     6 Jul 10 18:46 pg_stat_tmp
drwx------ 2 gpadmin gpadmin    18 Jul 10 18:46 pg_subtrans
drwx------ 2 gpadmin gpadmin     6 Jul 10 18:46 pg_tblspc
drwx------ 2 gpadmin gpadmin     6 Jul 10 18:46 pg_twophase
drwx------ 2 gpadmin gpadmin     6 Jul 10 18:47 pg_utilitymodedtmredo
-rw------- 1 gpadmin gpadmin     4 Jul 10 18:46 PG_VERSION
drwx------ 3 gpadmin gpadmin    60 Jul 10 18:46 pg_xlog
-rw------- 1 gpadmin gpadmin    88 Jul 10 18:46 postgresql.auto.conf
-rw------- 1 gpadmin gpadmin 23645 Jul 10 18:46 postgresql.conf
-rw------- 1 gpadmin gpadmin    95 Jul 10 18:47 postmaster.opts
-rw------- 1 gpadmin gpadmin    76 Jul 10 18:47 postmaster.pid
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
 dbid | content | role | preferred_role | mode | status | port |           hostname            |            address
       |          datadir
------+---------+------+----------------+------+--------+------+-------------------------------+-------------------------------+----------------------------
    1 |      -1 | p    | p              | n    | u      | 5432 | gpmaster.ru-central1.internal | gpmaster.ru-central1.internal | /home/gpadmin/data/gpseg-1
    5 |       3 | p    | p              | n    | u      | 6000 | gp1.ru-central1.internal      | gp1.ru-central1.internal      | /home/gpadmin/data/gpseg3
    8 |       6 | p    | p              | n    | u      | 6000 | gp2.ru-central1.internal      | gp2.ru-central1.internal      | /home/gpadmin/data/gpseg6
   11 |       9 | p    | p              | n    | u      | 6000 | gp3.ru-central1.internal      | gp3.ru-central1.internal      | /home/gpadmin/data/gpseg9
    2 |       0 | p    | p              | n    | u      | 6000 | gpmaster.ru-central1.internal | gpmaster.ru-central1.internal | /home/gpadmin/data/gpseg0
    6 |       4 | p    | p              | n    | u      | 6001 | gp1.ru-central1.internal      | gp1.ru-central1.internal      | /home/gpadmin/data/gpseg4
    9 |       7 | p    | p              | n    | u      | 6001 | gp2.ru-central1.internal      | gp2.ru-central1.internal      | /home/gpadmin/data/gpseg7
   12 |      10 | p    | p              | n    | u      | 6001 | gp3.ru-central1.internal      | gp3.ru-central1.internal      | /home/gpadmin/data/gpseg10
    3 |       1 | p    | p              | n    | u      | 6001 | gpmaster.ru-central1.internal | gpmaster.ru-central1.internal | /home/gpadmin/data/gpseg1
    7 |       5 | p    | p              | n    | u      | 6002 | gp1.ru-central1.internal      | gp1.ru-central1.internal      | /home/gpadmin/data/gpseg5
   10 |       8 | p    | p              | n    | u      | 6002 | gp2.ru-central1.internal      | gp2.ru-central1.internal      | /home/gpadmin/data/gpseg8
   13 |      11 | p    | p              | n    | u      | 6002 | gp3.ru-central1.internal      | gp3.ru-central1.internal      | /home/gpadmin/data/gpseg11
    4 |       2 | p    | p              | n    | u      | 6002 | gpmaster.ru-central1.internal | gpmaster.ru-central1.internal | /home/gpadmin/data/gpseg2
(13 rows)
</pre>

#### тестовые данные
<pre>
1.22)

генерируем числа от одного до 10 миллионов и помещаем в нашу таблицу
create table t1 as select generate_series(1,10000000) as colA distributed by (colA);


проверяем что у нас десять миллионов значений и за какое количество времени выполнился 
test=# EXPLAIN(ANALYZE) select count(*) from t1;
                                                             QUERY PLAN

-------------------------------------------------------------------------------------------------------------------------------------
 Aggregate  (cost=0.00..448.13 rows=1 width=8) (actual time=592.211..592.211 rows=1 loops=1)
   ->  Gather Motion 12:1  (slice1; segments: 12)  (cost=0.00..448.13 rows=1 width=8) (actual time=201.497..592.181 rows=12 loops=1)
         ->  Aggregate  (cost=0.00..448.13 rows=1 width=8) (actual time=111.944..111.945 rows=1 loops=1)
               ->  Seq Scan on t1  (cost=0.00..446.58 rows=833334 width=1) (actual time=0.015..297.044 rows=834529 loops=1)
 Planning time: 30.997 ms
   (slice0)    Executor memory: 59K bytes.
   (slice1)    Executor memory: 58K bytes avg x 12 workers, 58K bytes max (seg0).
 Memory used:  128000kB
 Optimizer: Pivotal Optimizer (GPORCA)
 Execution time: 592.626 ms
(10 rows)
</pre>

#### EXPLAIN(ANALYZE) select sum(colA) from t1
<pre>
1.23)

test=# EXPLAIN(ANALYZE) select sum(colA) from t1;
                                                             QUERY PLAN
-------------------------------------------------------------------------------------------------------------------------------------
 Aggregate  (cost=0.00..453.16 rows=1 width=8) (actual time=540.148..540.149 rows=1 loops=1)
   ->  Gather Motion 12:1  (slice1; segments: 12)  (cost=0.00..453.16 rows=1 width=8) (actual time=340.690..540.126 rows=12 loops=1)
         ->  Aggregate  (cost=0.00..453.16 rows=1 width=8) (actual time=474.825..474.825 rows=1 loops=1)
               ->  Seq Scan on t1  (cost=0.00..446.58 rows=833334 width=4) (actual time=0.008..257.992 rows=834529 loops=1)
 Planning time: 2.499 ms
   (slice0)    Executor memory: 59K bytes.
   (slice1)    Executor memory: 58K bytes avg x 12 workers, 58K bytes max (seg0).
 Memory used:  128000kB
 Optimizer: Pivotal Optimizer (GPORCA)
 Execution time: 540.501 ms
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

new
Идентификатор ключа:
YCAJEUqYD2tU3jgU2_jXoYImx

Ваш секретный ключ:
YCPQB68gMTtEmWhj-UQNp7HvY85KZFkBYdWL1lHP
Сохраните идентификатор и ключ. После закрытия диалога значение ключа будет недоступно.

</pre>

#### права доступа к моему бакету
<pre>
2.7)

добавил права доступа к моему бакету myotus для моего сервисного аакаунта myvorori
</pre>

#### s3fs-fuse
<pre>
2.8) 

создаю каталог для данных и устанавливаю s3fs-fuse
yum install s3fs-fuse
su - gpadmin
mkdir /home/gpadmin/taxi
cd /home/gpadmin/taxi
</pre>

#### .passwd-s3fs
<pre>
2.9)

добавляю ранее созданный индификатор,двоеточие и сервисный ключ
/home/gpadmin/.passwd-s3fs
echo YCAJEUqYD2tU3jgU2_jXoYImx:YCPQB68gMTtEmWhj-UQNp7HvY85KZFkBYdWL1lHP > /home/gpadmin/.passwd-s3fs


ПРОВЕРЯЮ!!!!!!!!!
cat /home/gpadmin/.passwd-s3fs
YCAJEUqYD2tU3jgU2_jXoYImx:YCPQB68gMTtEmWhj-UQNp7HvY85KZFkBYdWL1lHP

chmod 600 /home/gpadmin/.passwd-s3fs
chmod 600 /home/gpadmin/.passwd-s3fs
chmod 600 /home/gpadmin/.passwd-s3fs
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
проверяю появился новый подключенный раздел s3fs          
[gpadmin@gpmaster taxi]$ df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        1.9G     0  1.9G   0% /dev
tmpfs           1.9G   56K  1.9G   1% /dev/shm
tmpfs           1.9G  720K  1.9G   1% /run
tmpfs           1.9G     0  1.9G   0% /sys/fs/cgroup
/dev/vda2        20G  2.9G   18G  15% /
tmpfs           379M     0  379M   0% /run/user/1000
tmpfs           379M     0  379M   0% /run/user/997
tmpfs           379M     0  379M   0% /run/user/0
s3fs            4.0G     0  4.0G   0% /home/gpadmin/taxi

#### проверяю данные
<pre>
2.12) 

видим наши данные в нашем подгруженном бакете
[gpadmin@gpmaster taxi]$ ls -l /home/gpadmin/taxi
total 6782364
-rw-r----- 1 gpadmin gpadmin 267363203 Jul 10 19:25 taxi.csv.000000000000
-rw-r----- 1 gpadmin gpadmin 267344607 Jul 10 19:49 taxi.csv.000000000001
-rw-r----- 1 gpadmin gpadmin 267390243 Jul 10 19:35 taxi.csv.000000000002
-rw-r----- 1 gpadmin gpadmin 267417696 Jul 10 19:49 taxi.csv.000000000003
-rw-r----- 1 gpadmin gpadmin 266371749 Jul 10 19:43 taxi.csv.000000000004
-rw-r----- 1 gpadmin gpadmin 266233035 Jul 10 19:48 taxi.csv.000000000005
-rw-r----- 1 gpadmin gpadmin 266219968 Jul 10 19:25 taxi.csv.000000000006
-rw-r----- 1 gpadmin gpadmin 267380379 Jul 10 19:25 taxi.csv.000000000007
-rw-r----- 1 gpadmin gpadmin 267367985 Jul 10 19:38 taxi.csv.000000000008
-rw-r----- 1 gpadmin gpadmin 267278191 Jul 10 19:35 taxi.csv.000000000009
-rw-r----- 1 gpadmin gpadmin 267389474 Jul 10 19:25 taxi.csv.000000000010
-rw-r----- 1 gpadmin gpadmin 267331103 Jul 10 19:25 taxi.csv.000000000011
-rw-r----- 1 gpadmin gpadmin 266153465 Jul 10 19:37 taxi.csv.000000000012
-rw-r----- 1 gpadmin gpadmin 267368711 Jul 10 19:35 taxi.csv.000000000013
-rw-r----- 1 gpadmin gpadmin 265622273 Jul 10 19:38 taxi.csv.000000000014
-rw-r----- 1 gpadmin gpadmin 267319434 Jul 10 19:49 taxi.csv.000000000015
-rw-r----- 1 gpadmin gpadmin 267311996 Jul 10 19:30 taxi.csv.000000000016
-rw-r----- 1 gpadmin gpadmin 267325169 Jul 10 19:44 taxi.csv.000000000017
-rw-r----- 1 gpadmin gpadmin 267373075 Jul 10 19:49 taxi.csv.000000000018
-rw-r----- 1 gpadmin gpadmin 267377563 Jul 10 19:48 taxi.csv.000000000019
-rw-r----- 1 gpadmin gpadmin 267374748 Jul 10 19:44 taxi.csv.000000000020
-rw-r----- 1 gpadmin gpadmin 267391164 Jul 10 19:43 taxi.csv.000000000021
-rw-r----- 1 gpadmin gpadmin 267335900 Jul 10 19:17 taxi.csv.000000000022
-rw-r----- 1 gpadmin gpadmin 267348008 Jul 10 19:38 taxi.csv.000000000023
-rw-r----- 1 gpadmin gpadmin 267374332 Jul 10 19:30 taxi.csv.000000000024
-rw-r----- 1 gpadmin gpadmin 267369255 Jul 10 19:30 taxi.csv.000000000025
</pre>

#### создаем тестовую бд и таблицу
<pre>
2.13) 

создаем бд и таблицу
Загрузим данные в БД test:
psql -U gpadmin postgres -p 5432 -h gpmaster.ru-central1.internal
psql -U gpadmin postgres -p 5432 -h gpmaster.ru-central1.internal
psql -U gpadmin postgres -p 5432 -h gpmaster.ru-central1.internal


\c test

Чтобы обеспечить равномерное распределение данных, вы хотите выбрать ключ распределения, уникальный для каждой записи, или, если это 
невозможно, выберите РАСПРЕДЕЛЕНИЕ СЛУЧАЙНО. Например:

CREATE TABLE taxi_trips0(unique_key text
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
DISTRIBUTED BY (unique_key);


create table taxi_trips2 (
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
for f in *.csv*; do psql -U gpadmin -p 5432 -h gpmaster.ru-central1.internal -d test -c "\\COPY taxi_trips FROM PROGRAM 'cat $f' CSV HEADER"; done
for f in *.csv*; do psql -U gpadmin -p 5432 -h gpmaster.ru-central1.internal -d test -c "\\COPY taxi_trips FROM PROGRAM 'cat $f' CSV HEADER"; done
for f in *.csv*; do psql -U gpadmin -p 5432 -h gpmaster.ru-central1.internal -d test -c "\\COPY taxi_trips FROM PROGRAM 'cat $f' CSV HEADER"; done
[gpadmin@gpmaster taxi]$ for f in *.csv*; do psql -U gpadmin -p 5432 -h gpmaster.ru-central1.internal -d test -c "\\COPY taxi_trips FROM PROGRAM 'cat $f' CSV HEADER"; done
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


#### прогнал запросы 

<pre>
3.2)
---------------------------------------------------------------------------------------------------------------------------------------
EXPLAIN(ANALYZE) SELECT payment_type, round(sum(tips))/round(sum(trip_total)*100) + 0 as tips_percent, count(*) as c 
FROM taxi_trips
group by payment_type
order by c;

на gp 
Execution time: 36975.560 ms

на Postgre
Execution Time: 208397.652 ms
---------------------------------------------------------------------------------------------------------------------------------------
</pre>


#### тестовая генерация данных
<pre>
3.3)

---------------------------------------------------------------------------------------------------------------------------------------
тестовая генерация данных:
EXPLAIN(ANALYZE) create table t14 as select generate_series(1,1000000) as colA distributed by (colA);

на gp 
Execution time: 804.079 ms

на ванильном Postgre
Execution time: 3.981 sec
---------------------------------------------------------------------------------------------------------------------------------------
</pre>

#### select count(*) from t14;
<pre>
3.4)

---------------------------------------------------------------------------------------------------------------------------------------
запрос:
EXPLAIN(ANALYZE) select count(*) from t14;

на gp
Execution time: 182.096 ms

на ванильном Postgre
Execution Time: 114.369 ms
---------------------------------------------------------------------------------------------------------------------------------------
</pre>

#### EXPLAIN(ANALYZE) select sum(colA) from t14;
<pre>
3.5)

---------------------------------------------------------------------------------------------------------------------------------------
запрос:
EXPLAIN(ANALYZE) select sum(colA) from t14;

на gp
Execution time: 79.044 ms

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
каждый раз когда сталкиваюсь s3fs напарываюсь на какието фантомные боли то с правами что то не так то с ключем для сервисного акаунта проблема
вроде бы все окей но нет ЯО не пускает ) надо все почистить и заного нагенерить) выдать права пройти все круги.....

с gp очень все интересно и архитектура и решение работы postgres под капотом . будем пробывать практиковаться еше один новый космос который требуется шупать
буду настраивать свой локальный кластер на vm-ках чтобы поиграться с небольшим объемом данных и углубленно понять каконо работает тема ну оочень обширная и интересная
</pre>




