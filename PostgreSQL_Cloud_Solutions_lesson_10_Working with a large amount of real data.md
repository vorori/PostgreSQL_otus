#### 1)

#### Устанаввливаю настраиваю Greenplum

<pre>
</pre>



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


2)
cat /etc/redhat-release
cat /proc/cpuinfo | grep processor
cat /proc/meminfo  | grep MemTotal


3)
настройка ОС
vim /etc/sysctl.conf
vm.overcommit_memory = 2
vm.overcommit_ratio = 95 
net.ipv4.ip_local_port_range = 10000 65535 
kernel.sem = 250 2048000 200 8192
sysctl --system


4)
создать столько файлов и процессов вплодь до этого числа
vim /etc/security/limits.conf 
* soft nofile 524288
* hard nofile 524288
* soft nproc 131072
* hard nproc 131072


5)
добавляем групу gpadmin
groupadd gpadmin
useradd gpadmin -r -m -g gpadmin
echo Q123456789q | passwd --stdin gpadmin


6)
открываю порты если разворачиваемся не на яндекс облаке
firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="10.129.0.13" port protocol="tcp" port="22" accept"
firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="10.129.0.31" port protocol="tcp" port="22" accept"
firewall-cmd --reload		
firewall-cmd --list-all		


7)
скачиваем устанавливаем
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




#### 2)

#### Загрузить в неё данные (от 10 до 100 Гб)

<pre>
</pre>

#### 3)

#### Сравнить скорость выполнения запросов на PosgreSQL и выбранной СУБД


<pre>
</pre>


#### 4)

#### Описать что и как делали и с какими проблемами столкнулись

<pre>
</pre>