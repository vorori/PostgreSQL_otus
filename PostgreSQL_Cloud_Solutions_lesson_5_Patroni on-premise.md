
### Создаем 3 ВМ для etcd + 3 ВМ для Patroni +1 HA proxy (при проблемах можно на 1 хосте развернуть)
### Инициализируем кластер
### Проверяем отказоустойсивость
### *настраиваем бэкапы через wal-g или pg_probackup


# 1)

# Создаем 3 ВМ для etcd 


<pre> 
Отключить Selinux (необязательно)
Если вы знакомы с SELINUX, вам следует настроить его соответствующим образом. Если у вас нет большого опыта, рекомендуется отключить selinux, чтобы избежать каких-либо трудностей при настройке кластера.
 
Войдите в каждый свой узел с правами пользователя sudo без полномочий root и отредактируйте файл /etc/selinux/config в любом из ваших любимых текстовых редакторов:
 
vim /etc/selinux/config
Измените SELINUX=enforcing на SELINUX=disabled

SELINUX=disabled


Перезагрузите все узлы, чтобы изменения selinux вступили в силу:
 
sudo shutdown -r now



sudo firewall-cmd --zone=public --add-port=5432/tcp --permanent
sudo firewall-cmd --zone=public --add-port=6432/tcp --permanent
sudo firewall-cmd --zone=public --add-port=8008/tcp --permanent
sudo firewall-cmd --zone=public --add-port=2379/tcp --permanent
sudo firewall-cmd --zone=public --add-port=2380/tcp --permanent
sudo firewall-cmd --permanent --zone=public --add-service=http
sudo firewall-cmd --zone=public --add-port=5000/tcp --permanent
sudo firewall-cmd --zone=public --add-port=5001/tcp --permanent
sudo firewall-cmd --zone=public --add-port=7000/tcp --permanent
sudo firewall-cmd --add-rich-rule='rule protocol value="vrrp" accept' --permanent
sudo firewall-cmd --reload

sudo firewall-cmd --reload
firewall-cmd --list-all

часть 1
==========================================================================================================================================================================
1)
создал сеть остнастка диспетчер виртуальных комутаторов "VM and PC" для обединения рабочей станции с серверами 
сеть 192.168.1.0 маска 255.255.255.0


2)
создал 1 родительский диск 


3)
создал 4 диска разностных


4)
создал 4vm
otus_ETCD_vm1
otus_ETCD_vm2
otus_ETCD_vm3


5)
Запускаем каждый сервер и делаем настройки
настройка сеть на каждом сервере linux 


vim /etc/sysconfig/network-scripts/ifcfg-eth0
systemctl restart network


6) по итогу получили список серверов
ТЕСТ_local_otus_ETCD_vm1_centos          192.168.1.14
ТЕСТ_local_otus_ETCD_vm2_centos          192.168.1.15
ТЕСТ_local_otus_ETCD_vm3_centos          192.168.1.16



==========================================================================================================================================================================



часть 2
==========================================================================================================================================================================



1)
Установка "etcd"
yum install etcd-3.3.11-2.el7.centos.x86_64.rpm


2)
/usr/bin/etcd --version

/usr/bin/etcd --help
/usr/bin/etcd --help
/usr/bin/etcd --help

документация:
https://etcd.io/docs/v3.3/
https://etcd.io/docs/v3.3/
https://etcd.io/docs/v3.3/

etcd Version: 3.3.11
Git SHA: 2cf9e51
Go Version: go1.10.3
Go OS/Arch: linux/amd64



5)
Создаём конфигурацию начального/мастер узла кластера для всех трех нод меняю соответствующие значения:
vim /etc/etcd/etcd.conf

#[Member]
ETCD_DATA_DIR="/var/lib/etcd"
ETCD_LISTEN_PEER_URLS="http://0.0.0.0:2380"
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
ETCD_NAME="srv01"
ETCD_HEARTBEAT_INTERVAL="1000"
ETCD_ELECTION_TIMEOUT="5000"

#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.1.14:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.1.14:2379"
ETCD_INITIAL_CLUSTER="srv01=http://192.168.1.14:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd_cluster_777"
ETCD_INITIAL_CLUSTER_STATE="new"






6)
Запускаем "etcd" службу и открываем порты:
systemctl enable etcd
systemctl start etcd
systemctl stop etcd

systemctl start etcd
systemctl status etcd




описание параметров:
https://etcd.io/docs/v3.3/op-guide/configuration/

Исходное состояние кластера  (“new” (новый) or “existing” (существующий). )
Установите new для всех участников, присутствующих во время начальной статической или DNS-загрузки. 
Если для этой опции установлено значение existing, etcd попытается присоединиться к существующему кластеру. 
Если установлено неправильное значение, etcd попытается запуститься, но завершится ошибкой.

значение по умолчанию default: “new”
пременная : ETCD_INITIAL_CLUSTER_STATE
-----------------------------------------


открываю порты на певой ноде
------------------------------------------------------------------------------------------------------------------------------------------------
firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.1.6" port protocol="tcp" port="2380" accept"
firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.1.6" port protocol="tcp" port="2379" accept"

firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.1.15" port protocol="tcp" port="2380" accept"
firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.1.15" port protocol="tcp" port="2379" accept"

firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.1.16" port protocol="tcp" port="2380" accept"
firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.1.16" port protocol="tcp" port="2379" accept"

firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.1.17" port protocol="tcp" port="2380" accept"
firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.1.17" port protocol="tcp" port="2379" accept"

firewall-cmd --reload		
firewall-cmd --list-all		
------------------------------------------------------------------------------------------------------------------------------------------------

7)
Проверяем статус службы, в случае без ошибочного статуса, включаем службу "etcd" в автозапуск:
[root@localhost etcd]# systemctl status etcd
● etcd.service - Etcd Server
   Loaded: loaded (/usr/lib/systemd/system/etcd.service; enabled; vendor preset: disabled)
   Active: active (running) since Thu 2023-02-02 14:11:14 MSK; 2s ago
 Main PID: 13839 (etcd)
   CGroup: /system.slice/etcd.service
           └─13839 /usr/bin/etcd --name=srv01 --data-dir=/var/lib/etcd --listen-client-urls=http://0.0.0.0:2379

Feb 02 14:11:14 localhost.localdomain etcd[13839]: 7aa04f7c10ace564 received MsgVoteResp from 7aa04f7c10ace564 at term 2
Feb 02 14:11:14 localhost.localdomain etcd[13839]: 7aa04f7c10ace564 became leader at term 2
Feb 02 14:11:14 localhost.localdomain etcd[13839]: raft.node: 7aa04f7c10ace564 elected leader 7aa04f7c10ace564 at term 2
Feb 02 14:11:14 localhost.localdomain etcd[13839]: setting up the initial cluster version to 3.3
Feb 02 14:11:14 localhost.localdomain etcd[13839]: published {Name:srv01 ClientURLs:[http://192.168.1.14:2379]} to cluster c60e45ce58a3920
Feb 02 14:11:14 localhost.localdomain etcd[13839]: ready to serve client requests
Feb 02 14:11:14 localhost.localdomain etcd[13839]: serving insecure client requests on [::]:2379, this is strongly discouraged!
Feb 02 14:11:14 localhost.localdomain systemd[1]: Started Etcd Server.
Feb 02 14:11:14 localhost.localdomain etcd[13839]: set the initial cluster version to 3.3
Feb 02 14:11:14 localhost.localdomain etcd[13839]: enabled capabilities for version 3.3


8)
Проверяем состояние кластера:
etcdctl cluster-health
member 7aa04f7c10ace564 is healthy: got healthy result from http://192.168.1.14:2379
cluster is healthy
[root@localhost ~]#



9)
Добавляем второй узел выполняем команну на добавление второго узала на первой ноде выполняю команду на присоединение!!!:
etcdctl member add srv02 http://192.168.1.15:2380
Added member named srv02 with ID fb05c721143a1777 to cluster

ETCD_NAME="srv02"
ETCD_INITIAL_CLUSTER="srv01=http://192.168.1.14:2380,srv02=http://192.168.1.15:2380"
ETCD_INITIAL_CLUSTER_STATE="existing"



открываю порты на второй ноде
------------------------------------------------------------------------------------------------------------------------------------------------
firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.1.6" port protocol="tcp" port="2380" accept"
firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.1.6" port protocol="tcp" port="2379" accept"

firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.1.14" port protocol="tcp" port="2380" accept"
firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.1.14" port protocol="tcp" port="2379" accept"

firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.1.16" port protocol="tcp" port="2380" accept"
firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.1.16" port protocol="tcp" port="2379" accept"

firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.1.17" port protocol="tcp" port="2380" accept"
firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.1.17" port protocol="tcp" port="2379" accept"

firewall-cmd --reload		
firewall-cmd --list-all		
------------------------------------------------------------------------------------------------------------------------------------------------

10)
Устанавливаем в аналогичном порядке "etcd" на втором узле, но изменяем конфигурацию под атрибуты данного узла:
vim /etc/etcd/etcd.conf

#[Member]
ETCD_DATA_DIR="/var/lib/etcd"
ETCD_LISTEN_PEER_URLS="http://0.0.0.0:2380"
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
ETCD_NAME="srv02"                                                                            # меняю на имя второго узла "srv02"  
ETCD_HEARTBEAT_INTERVAL="1000"
ETCD_ELECTION_TIMEOUT="5000"

#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.1.15:2380"                                  # меняю на ip второго узла "http://192.168.1.15:2380"  
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.1.15:2379"                                        # меняю на ip второго узла "http://192.168.1.15:2379"  
ETCD_INITIAL_CLUSTER="srv01=http://192.168.1.14:2380,srv02=http://192.168.1.15:2380"         # сюда вставляю "srv01=http://192.168.1.14:2380,srv02=http://192.168.1.15:2380" 
ETCD_INITIAL_CLUSTER_TOKEN="etcd_cluster_777"                                                # имя кластера остается прежнее "etcd_cluster_777" 
ETCD_INITIAL_CLUSTER_STATE="existing"                                                        #"Обратите внимание на флаг, он отличается так же"


11)
Запускаем службу и проверяем состав кластера на состояние если все ок добавляю в автозагрузку:
systemctl enable etcd
systemctl start etcd
systemctl stop etcd
systemctl status etcd

etcdctl cluster-health
member 7aa04f7c10ace564 is healthy: got healthy result from http://192.168.1.14:2379
member fb05c721143a1777 is healthy: got healthy result from http://192.168.1.15:2379
cluster is healthy




12)
etcdctl cluster-health
member 7aa04f7c10ace564 is healthy: got healthy result from http://192.168.1.14:2379
member fb05c721143a1777 is healthy: got healthy result from http://192.168.1.15:2379
cluster is healthy

12.1)
моменял значение на певой ноде 
на ETCD_INITIAL_CLUSTER="srv01=http://192.168.1.14:2380,srv02=http://192.168.1.15:2380"
и ерезапустил службу проверил что все хорошо кластер на связи
[root@localhost etcd]# etcdctl cluster-health
member 7aa04f7c10ace564 is healthy: got healthy result from http://192.168.1.14:2379
member fb05c721143a1777 is healthy: got healthy result from http://192.168.1.15:2379

12.2)
добавляю третью ноду
на первой ноде выполняю команду на присоединение
etcdctl member add srv03 http://192.168.1.16:2380
Added member named srv03 with ID 69dad9f443ce9230 to cluster

ETCD_NAME="srv03"
ETCD_INITIAL_CLUSTER="srv03=http://192.168.1.16:2380,srv01=http://192.168.1.14:2380,srv02=http://192.168.1.15:2380"
ETCD_INITIAL_CLUSTER_STATE="existing"




открываю порты на третей ноде
------------------------------------------------------------------------------------------------------------------------------------------------
firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.1.6" port protocol="tcp" port="2380" accept"
firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.1.6" port protocol="tcp" port="2379" accept"

firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.1.14" port protocol="tcp" port="2380" accept"
firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.1.14" port protocol="tcp" port="2379" accept"

firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.1.15" port protocol="tcp" port="2380" accept"
firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.1.15" port protocol="tcp" port="2379" accept"

firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.1.17" port protocol="tcp" port="2380" accept"
firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.1.17" port protocol="tcp" port="2379" accept"

firewall-cmd --reload		
firewall-cmd --list-all		
------------------------------------------------------------------------------------------------------------------------------------------------



12.3)
меняю конфиг под третью ноду на третей ноде
vim /etc/etcd/etcd.conf


#[Member]
ETCD_DATA_DIR="/var/lib/etcd"
ETCD_LISTEN_PEER_URLS="http://0.0.0.0:2380"
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
ETCD_NAME="srv03"
ETCD_HEARTBEAT_INTERVAL="1000"
ETCD_ELECTION_TIMEOUT="5000"

#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.1.16:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.1.16:2379"
ETCD_INITIAL_CLUSTER="srv03=http://192.168.1.16:2380,srv01=http://192.168.1.14:2380,srv02=http://192.168.1.15:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd_cluster_777"
ETCD_INITIAL_CLUSTER_STATE="existing"




12.4)
Запускаем службу и проверяем состав кластера на состояние если все ок добавляю в автозагрузку:
systemctl enable etcd
systemctl start etcd
systemctl stop etcd
systemctl status etcd

etcdctl cluster-health
member 69dad9f443ce9230 is healthy: got healthy result from http://192.168.1.16:2379
member 7aa04f7c10ace564 is healthy: got healthy result from http://192.168.1.14:2379
member fb05c721143a1777 is healthy: got healthy result from http://192.168.1.15:2379
cluster is healthy


ОБЯЗАТЕЛЬНО ВСЕ НОДЫ ПРИВЕСТИ К ЕДИНОМУ ВИДУ
Порядок дальнейшего добавления узлов в кластер опускаю, он идентичен на добавлением второго узла и после добавления Всех необходимых узлов, 
нужно привести конфигурацию на всех участниках к единому виду по следующим переменным:
ETCD_INITIAL_CLUSTER="srv03=http://192.168.1.16:2380,srv01=http://192.168.1.14:2380,srv02=http://192.168.1.15:2380"
ETCD_INITIAL_CLUSTER_STATE="existing"





13.1)

если надор пресоздать ноду
-----------------------------------------
systemctl stop etcd
rm -rf /var/lib/etcd/member/*
etcdctl cluster-health
etcdctl member list
etcdctl member remove 75f844b9fc38df9f  #Удаляем проблемную



Возврашаем потерянную ноду etcd в строй
systemctl stop etcd

На живой ноде кластера проверяем состояние кластера (видим нашу проблемную ноду, и что кластер встатусе degraded)
etcdctl cluster-health
Выводим список нод с их данными подключения и ID

etcdctl member list
Удаляем проблемную ноду из кластера по ее ID
(в данном конкретном случае: 7aa04f7c10ace564:):

(ввожу команду на удаления мембера на ноде лидере)
(после ввода команды попросит веести пароль для пользователя root в кластере etcd)
etcdctl -u root member remove 6d6e80d1f39b162f
etcdctl -u root member remove 7aa04f7c10ace564
etcdctl -u root member remove 7aa04f7c10ace564

Теперь добавим ноду обратно в кластер, указав ее имя и
URL для подключения. Нода будет добавленаи получит в кластере новый ID
(в данном случае: 9f86b6b7b23da7e8):

etcdctl -u root member add srv01 http://192.168.1.14:2380
etcdctl -u root member add srv01 http://192.168.1.14:2380
etcdctl -u root member add srv01 http://192.168.1.14:2380

etcdctl member list
На проблемной ноде редактируем конфиг /etc/etcd/etcd.conf, прописывая в настройке
ETCD_INITIAL_CLUSTER_STATE «existing» вместо «new

ETCD_INITIAL_CLUSTER_STATE="existing"
В том же конфиге /etc/etcd/etcd.conf смотрим где нода хранит данные ectd в параметре

ETCD_DATA_DIR (в нашем случае ETCD_DATA_DIR=»/var/lib/etcd»), переходим в эту папку и удаляем из ее поддиректории member все содержимое:

rm -rf /var/lib/etcd/member/*

Запускаем на проблемной ноде службу etcd. Нода должна подключиться к существующему кластеру,получить выданный ей кластером ID
(в данном случае: 71fbb082f6f4f045) и синхронизироваться. Все этоможно проверить выполнив:

systemctl stop etcd
systemctl start etcd
systemctl status etcd
Теперь проверяем что наши ноды в кластере и что кластер теперь в нормальном состоянии:

etcdctl member list
etcdctl cluster-health



rm -rf /var/lib/etcd/member/snap/*
rm -rf /var/lib/etcd/member/wal/*


13.2)
https://etcd.io/docs/v2.3/authentication/
https://etcd.io/docs/v2.3/authentication/
https://etcd.io/docs/v2.3/authentication/

аутентификацию не включал 
аутентификацию не включал 
аутентификацию не включал 

Далее нам необходимо создать пользователя кластера с высокими правами, для внутренней смежной коммуникацией между "patroni" и кластером "etcd":
etcdctl user add root
New password: 
User root created

etcdctl user get root
User: root
Roles:  root

etcdctl auth enable
Authentication Enabled

etcdctl --username root user get root
Password: 
User: root
Roles:  root

</pre>


# 2) создаем 3 ВМ для Patroni

<pre> 
часть 2 Установка и настройка сервиса "patroni" на "Centos8"
==========================================================================================================================================================================
==========================================================================================================================================================================
==========================================================================================================================================================================
==========================================================================================================================================================================
==========================================================================================================================================================================


1)
install postgre14 в первую очердь

install python и тд 

патрони устанавливаем в последнюю очередь

sudo dnf install --disablerepo='*' *.rpm
sudo dnf install --disablerepo='*' *.rpm
sudo dnf install --disablerepo='*' *.rpm


2)
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
если есть ошибка Error: Failed to download metadata for repo 'AppStream'
и нам не нужны установки с репозиториев

[root@localhost ~]# cd /etc/yum.repos.d/
[root@localhost yum.repos.d]# sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
[root@localhost yum.repos.d]# sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*
yum update -y

а потом 
sudo dnf install --disablerepo='*' *.rpm
sudo dnf install --disablerepo='*' *.rpm
sudo dnf install --disablerepo='*' *.rpm
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

3)
--------------------------------------------------
Устанавливаем "patroni" если есть интернет то:
patroni --version
patroni 2.1.7
--------------------------------------------------

#обязательно выключаем postgres из автозагрузки
sudo systemctl disable postgresql-14
sudo systemctl disable postgresql-14
sudo systemctl disable postgresql-14

#проверяем доступность postgres
netstat -antp | grep post
netstat -antp | grep post
netstat -antp | grep post

#Проверяем успешность запуска
ss -ltn | grep 5432
ss -ltn | grep 5432
ss -ltn | grep 5432


4)
Настраиваем первый узел "node1", настроем окружение для службы "patroni":
sudo mkdir -p /etc/patroni/
sudo chown -R  postgres:postgres /etc/patroni/
chmod 700 /etc/patroni

5)
Создаём файл настроек для службы "patroni", файл имеет мои собственные параметры для моего окружения, 
мною было принято решение оставить их для наглядности того, в каком блоке необходимо выполнять параметризацию 
"PostgreSQL" посредством "patroni", описание/понимание каждой строки как в этом так и в предыдущих примерах, 
оставляю на совесть испытуемого, большинство настроек имеет конфигурацию по умолчанию, то как они были указаны в инструкциях от 1с и других источников:

Create the patroni.yml configuration file.
su postgres
vim /etc/patroni/patroni.yml
    /etc/patroni/patroni.yml
	
ОБЯЗАТЕЛЬНО СНОВА ВЫПОЛНИТЬ ЭТУ КОМАНДУ ПОСЛЕ СОЗДАНИЯ КОНФИГА ПАТРОНИ!!!
sudo chown -R  postgres:postgres /etc/patroni/
sudo chown -R  postgres:postgres /etc/patroni/
sudo chown -R  postgres:postgres /etc/patroni/


---------------------------------
в настройках службы
В приведённых настройках запуска особенно важны два параметра
TimeoutSec - время в секундах, которое система будет ожидать при запуске и остановке сервиса, перед тем как произвести попытку его внештатного завершения.
Restart - может принимать значения: no, on-success, on-failure, on-abnormal, on-watchdog, on-abort, или always. 
Определяет политику перезапуска сервиса в случае, если он завершает работу не по команде от systemd.
---------------------------------


vim
прежде чем вставлять что-либо, попробуйте использовать
:set paste



6)
Создаём сервис для запуска службы "patroni":
служба уже создана тк. устанавливали из пакетов


7)
Запускаем службу "patroni" и открываем порты:
systemctl daemon-reload

sudo systemctl daemon-reload
sudo systemctl enable patroni.service
sudo systemctl start patroni.service
sudo systemctl status patroni.service
sudo systemctl stop patroni.service

systemctl start patroni.service

patronictl -c /etc/patroni/patroni.yml list
patronictl -c /etc/patroni/patroni.yml list
patronictl -c /etc/patroni/patroni.yml list


после старта всех трех нод
patronictl -c /etc/patroni/patroni.yml list
+ Cluster: mycluster (7199603018565937232) ---+----+-----------+
| Member   | Host         | Role    | State   | TL | Lag in MB |
+----------+--------------+---------+---------+----+-----------+
| pg_node1 | 192.168.1.22 | Leader  | running |  1 |           |
| pg_node2 | 192.168.1.23 | Replica | running |  1 |         0 |
| pg_node3 | 192.168.1.24 | Replica | running |  1 |         0 |
+----------+--------------+---------+---------+----+-----------+


-----------------------------------
ЕСЛИ НАДО ПРЕЧИТАТЬ НАСТРОЙКИ patroni

patronictl -c /etc/patroni/patroni.yml reload mycluster pg_node1
patronictl -c /etc/patroni/patroni.yml restart mycluster pg_node1
patronictl -c /etc/patroni/patroni.yml reload mycluster pg_node2
patronictl -c /etc/patroni/patroni.yml restart mycluster pg_node2
patronictl -c /etc/patroni/patroni.yml reload mycluster pg_node3
patronictl -c /etc/patroni/patroni.yml restart mycluster pg_node3
-----------------------------------



-----------------------------------
ЕСЛИ НАДО ВЫПОЛНИТЬ ПЛАНОВОЕ ПРЕКЛЮЧЕНИЕ НОДЫ ЛИДЕРА

patronictl -c /etc/patroni/patroni.yml failover
patronictl -c /etc/patroni/patroni.yml failover
patronictl -c /etc/patroni/patroni.yml failover

-----------------------------------
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
(Если у нас уже стартанул кластер но нам нужно пресоздать кластер с другим именем к примеру то там нужно сначало удалить стары кластер из etcd
в моей превоночальной конфигурации это делается это командой:

если нужно посмотреть что за конфигурации сохранены в etcd: 
etcdctl ls /
etcdctl ls /pg_cluster/postgres

etcdctl rm /pg_cluster --recursive
etcdctl rm /pg_cluster --recursive
etcdctl rm /pg_cluster --recursive

etcdctl rm /pg_cluster/postgresql0 --recursive
etcdctl rm /pg_cluster/postgresql0 --recursive
etcdctl rm /pg_cluster/postgresql0 --recursive

и конечно удалить все физические данные кластера 
rm -rf /var/lib/pgsql/14/data/*
rm -rf /var/lib/pgsql/14/data/*
rm -rf /var/lib/pgsql/14/data/*


пример:
эти данные взяты из конфигурации  /etc/patroni/patroni.yml
namespace: /pg_cluster/
name: postgresql0

а после уже сновым конфигом стартануть patroni !!!!!

!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!


!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!ЧТОБЫ ИЗМЕНИТЬ КОНФИГУРАЦИЮ КЛАСТЕРА PATRONI после того как кластер уже работает !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!



1)
_________________
Вариант номер 1 |
                |___________________________________________________________
меняем конфигурацию на ноде лидера и выполняем перезапуск узла за узлом    |
___________________________________________________________________________|
Если вы не хотите перезапускать весь кластер, у вас есть возможность перезапустить каждый узел по отдельности.
Имейте в виду, что сначала необходимо перезапустить узел-лидер, иначе изменение не вступит в силу.

-----------------------------------------------------------------------------
команда на изменение параметра:
patronictl -c /etc/patroni/patroni.yml edit-config
patronictl -c /etc/patroni/patroni.yml edit-config
patronictl -c /etc/patroni/patroni.yml edit-config
-----------------------------------------------------------------------------

-----------------------------------------------------------------------------
показать лидера и кворум
patronictl -c /etc/patroni/patroni.yml list
patronictl -c /etc/patroni/patroni.yml list
patronictl -c /etc/patroni/patroni.yml list
-----------------------------------------------------------------------------

-----------------------------------------------------------------------------
пример изменения конфигурации на примере параметра max_connections и log_line_prefix и log_statement:
patronictl -c /etc/patroni/patroni.yml edit-config
patronictl -c /etc/patroni/patroni.yml edit-config
patronictl -c /etc/patroni/patroni.yml edit-config

loop_wait: 10
master_start_timeout: 10
maximum_lag_on_failover: 1048576
postgresql:
  parameters:
    checkpoint_timeout: 30
    hot_standby: 'on'
    log_line_prefix: '%m [%p] %d %u %h (transaction ID %x)'
    log_statement: all
    max_connections: 101
    max_replication_slots: 5
    max_wal_senders: 5
    track_activity_query_size: 16384
    wal_keep_segments: 8
    wal_level: replica
  use_pg_rewind: true
  use_slots: true
retry_timeout: 10
ttl: 30
-----------------------------------------------------------------------------

2)
_________________
Вариант номер 2 |
                |_________________________________________________________________________________________________________________
Перезапустите весь кластер                                                                                                        |
В случае, если вы не хотите перезапускать узел за узлом и у вас есть вероятность простоя, также можно перезапустить весь кластер  | 
__________________________________________________________________________________________________________________________________|

-----------------------------------------------------------------------------
команда на изменение параметра:
patronictl -c /etc/patroni/patroni.yml edit-config
patronictl -c /etc/patroni/patroni.yml edit-config
patronictl -c /etc/patroni/patroni.yml edit-config
-----------------------------------------------------------------------------

-----------------------------------------------------------------------------
показать лидера и кворум:
patronictl -c /etc/patroni/patroni.yml list
patronictl -c /etc/patroni/patroni.yml list
patronictl -c /etc/patroni/patroni.yml list
-----------------------------------------------------------------------------

-----------------------------------------------------------------------------
пререстартовать кластер:
patronictl -c /etc/patroni/patroni.yml restart mycluster
patronictl -c /etc/patroni/patroni.yml restart mycluster
patronictl -c /etc/patroni/patroni.yml restart mycluster
-----------------------------------------------------------------------------


!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!ОБЯЗАТЕЛЬНО ЭТО КОНФИГУРАЦИЯ СДЕЛАНА ДЛЯ ТРЕЕХ НОД КЛАСТЕРА ETCD КЛАСТЕР!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!ДЛЯ PATRONI NODE1 IP 192.168.1.22 !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!ETCD CLUSTER 192.168.1.14:2379 192.168.1.15:2379 192.168.1.16:2379!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
Create the patroni.yml configuration file.
su postgres
vim /etc/patroni/patroni.yml
    /etc/patroni/patroni.yml
	
ОБЯЗАТЕЛЬНО СНОВА ВЫПОЛНИТЬ ЭТУ КОМАНДУ ПОСЛЕ СОЗДАНИЯ КОНФИГА ПАТРОНИ!!!
sudo chown -R  postgres:postgres /etc/patroni/
sudo chown -R  postgres:postgres /etc/patroni/
sudo chown -R  postgres:postgres /etc/patroni/

tail -f /var/log/messages
tail -f /var/log/messages
tail -f /var/log/messages

tail -f /var/lib/pgsql/14/data/log/postgresql-Thu.log
tail -f /var/lib/pgsql/14/data/log/postgresql-Thu.log
tail -f /var/lib/pgsql/14/data/log/postgresql-Thu.log


scope: mycluster
namespace: /pg_cluster/
name: pg_node1

restapi:
  listen: 0.0.0.0:8008
  connect_address: 192.168.1.22:8008


etcd:
  #Provide host to do the initial discovery of the cluster topology:
  hosts:
   - 192.168.1.14:2379
   - 192.168.1.15:2379
   - 192.168.1.16:2379



bootstrap:
  # этот раздел будет записан в Etcd:/<namespace>/<scope>/config после инициализации нового кластера
  # and all other cluster members will use it as a `global configuration` (и все остальные члены кластера будут использовать его как «глобальную конфигурацию».)
  dcs:
    ttl: 30
    loop_wait: 10
    retry_timeout: 10
    maximum_lag_on_failover: 1048576
    master_start_timeout: 10
    postgresql:
      use_pg_rewind: true
      use_slots: true
      parameters:
         wal_level: replica
         hot_standby: "on"
         wal_keep_segments: 8
         max_wal_senders: 5
         max_replication_slots: 5
         checkpoint_timeout: 30
#        wal_level: hot_standby
#        hot_standby: "on"
#        max_connections: 100
#        max_worker_processes: 8
#        wal_keep_segments: 8
#        max_wal_senders: 10
#        max_replication_slots: 10
#        max_prepared_transactions: 0
#        max_locks_per_transaction: 64
#        wal_log_hints: "on"
#        track_commit_timestamp: "off"
#        archive_mode: "on"
#        archive_timeout: 1800s
#        archive_command: mkdir -p ../wal_archive && test ! -f ../wal_archive/%f && cp %p ../wal_archive/%f
#        recovery_conf:
#        restore_command: cp ../wal_archive/%f %p

  # некоторые желаемые параметры для 'initdb'
  initdb:  # Примечание: это должен быть список (некоторые параметры требуют значений, другие являются переключателями)
  - encoding: UTF8
  # - data-checksums

  pg_hba:  # Добавьте следующие строки в pg_hba.conf после запуска 'initdb'
  - host replication replicator 127.0.0.1/32    md5
  - host replication replicator 192.168.1.22/32 md5  
  - host replication replicator 192.168.1.23/32 md5 
  - host replication replicator 192.168.1.24/32 md5
  - host all         rewind_user 192.168.1.22/32 md5
  - host all         rewind_user 192.168.1.23/32 md5
  - host all         rewind_user 192.168.1.24/32 md5
  - host replication all        127.0.0.1/32    md5

  # Некоторые дополнительные пользователи пользователей, которых необходимо создать после инициализации нового кластера
  users:
    admin:
      password: passadmin12356789
      options:
        - createrole
        - createdb

postgresql:
  listen: 0.0.0.0:5432
  connect_address: 192.168.1.22:5432
  data_dir: /var/lib/pgsql/14/data
  bin_dir: /usr/pgsql-14/bin
  config_dir: /var/lib/pgsql/14/data
  pgpass: /tmp/pgpass0
  authentication:
    replication:
      username: replicator
      password: rep-pass
    superuser:
      username: postgres
      password: postgres
    rewind:  # Не влияет на postgres 10 и ниже
      username: rewind_user
      password: rewind_password
  parameters:

    #------------------------------------------------------------------------------
    # CONNECTIONS AND AUTHENTICATION
    #------------------------------------------------------------------------------

    listen_addresses: '*'
    port: 5432

    #------------------------------------------------------------------------------
    # RESOURCE USAGE (except WAL)
    #------------------------------------------------------------------------------

    shared_buffers: 2048MB
    temp_buffers: 8MB
    work_mem: 4MB
    maintenance_work_mem: 512MB
    max_connections: 100
    dynamic_shared_memory_type: posix
    seq_page_cost: 1
    random_page_cost: 4
    cpu_operator_cost: 0.0025

    #------------------------------------------------------------------------------
    # REPORTING AND LOGGING
    #------------------------------------------------------------------------------

    # - Where to Log -
    log_destination: 'stderr'
    logging_collector: on
    log_timezone: 'Europe/Moscow'

    # - Locale and Formatting -

    datestyle: 'iso, dmy'
    timezone: 'Europe/Moscow'

    # - Kernel Resources -
    max_files_per_process: 10000
    commit_delay: 0

    # - Genetic Query Optimizer -
    from_collapse_limit: 8
    join_collapse_limit: 8

    #------------------------------------------------------------------------------
    # AUTOVACUUM
    #------------------------------------------------------------------------------

    autovacuum_max_workers: 4 
    vacuum_cost_limit: 200
    autovacuum_naptime: 10s
    autovacuum_vacuum_scale_factor: 0.01
    autovacuum_analyze_scale_factor: 0.003

    #------------------------------------------------------------------------------
    # LOCK MANAGEMENT
    #------------------------------------------------------------------------------

    max_locks_per_transaction: 64

    # - Shared Library Preloading -

    shared_preload_libraries: 'pg_stat_statements, pg_stat_kcache, pg_wait_sampling'

    #------------------------------------------------------------------------------
    # QUERY TUNING
    #------------------------------------------------------------------------------

    effective_cache_size: 4GB

    # - Checkpoints -

    checkpoint_completion_target: 0.9
    min_wal_size: 2000MB
    max_wal_size: 4000MB

    #------------------------------------------------------------------------------
    # WRITE-AHEAD LOG
    #------------------------------------------------------------------------------

    wal_buffers: 16MB

    # - Other Planner Options -

    default_statistics_target: 500

    # - Asynchronous Behavior -

    effective_io_concurrency: 200

    # - Planner Cost Constants -
    random_page_cost: 1.1

    # - Asynchronous Behavior -

    max_worker_processes: 8
    max_parallel_workers_per_gather: 2
    max_parallel_workers: 8
    max_parallel_maintenance_workers: 2

tags:
    nofailover: false
    noloadbalance: false
    clonefrom: false
    nosync: false
	
	
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!ОБЯЗАТЕЛЬНО ЭТО КОНФИГУРАЦИЯ СДЕЛАНА ДЛЯ ТРЕЕХ НОД КЛАСТЕРА ETCD КЛАСТЕР!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!ДЛЯ PATRONI NODE2 IP 192.168.1.23 !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!ETCD CLUSTER 192.168.1.14:2379 192.168.1.15:2379 192.168.1.16:2379!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
Create the patroni.yml configuration file.
su postgres
vim /etc/patroni/patroni.yml
    /etc/patroni/patroni.yml
	
ОБЯЗАТЕЛЬНО СНОВА ВЫПОЛНИТЬ ЭТУ КОМАНДУ ПОСЛЕ СОЗДАНИЯ КОНФИГА ПАТРОНИ!!!
sudo chown -R  postgres:postgres /etc/patroni/
sudo chown -R  postgres:postgres /etc/patroni/
sudo chown -R  postgres:postgres /etc/patroni/

tail -f /var/log/messages
tail -f /var/log/messages
tail -f /var/log/messages

tail -f /var/lib/pgsql/14/data/log/postgresql-Thu.log
tail -f /var/lib/pgsql/14/data/log/postgresql-Thu.log
tail -f /var/lib/pgsql/14/data/log/postgresql-Thu.log



scope: mycluster
namespace: /pg_cluster/
name: pg_node2

restapi:
  listen: 0.0.0.0:8008
  connect_address: 192.168.1.23:8008


etcd:
  #Provide host to do the initial discovery of the cluster topology:
  hosts:
   - 192.168.1.14:2379
   - 192.168.1.15:2379
   - 192.168.1.16:2379



bootstrap:
  # этот раздел будет записан в Etcd:/<namespace>/<scope>/config после инициализации нового кластера
  # and all other cluster members will use it as a `global configuration` (и все остальные члены кластера будут использовать его как «глобальную конфигурацию».)
  dcs:
    ttl: 30
    loop_wait: 10
    retry_timeout: 10
    maximum_lag_on_failover: 1048576
    master_start_timeout: 10
    postgresql:
      use_pg_rewind: true
      use_slots: true
      parameters:
         wal_level: replica
         hot_standby: "on"
         wal_keep_segments: 8
         max_wal_senders: 5
         max_replication_slots: 5
         checkpoint_timeout: 30
#        wal_level: hot_standby
#        hot_standby: "on"
#        max_connections: 100
#        max_worker_processes: 8
#        wal_keep_segments: 8
#        max_wal_senders: 10
#        max_replication_slots: 10
#        max_prepared_transactions: 0
#        max_locks_per_transaction: 64
#        wal_log_hints: "on"
#        track_commit_timestamp: "off"
#        archive_mode: "on"
#        archive_timeout: 1800s
#        archive_command: mkdir -p ../wal_archive && test ! -f ../wal_archive/%f && cp %p ../wal_archive/%f
#        recovery_conf:
#        restore_command: cp ../wal_archive/%f %p

  # некоторые желаемые параметры для 'initdb'
  initdb:  # Примечание: это должен быть список (некоторые параметры требуют значений, другие являются переключателями)
  - encoding: UTF8
  # - data-checksums

  pg_hba:  # Добавьте следующие строки в pg_hba.conf после запуска 'initdb'
  - host replication replicator 127.0.0.1/32    md5
  - host replication replicator 192.168.1.22/32 md5  
  - host replication replicator 192.168.1.23/32 md5 
  - host replication replicator 192.168.1.24/32 md5
  - host all         rewind_user 192.168.1.22/32 md5
  - host all         rewind_user 192.168.1.23/32 md5
  - host all         rewind_user 192.168.1.24/32 md5
  - host replication all        127.0.0.1/32    md5

  # Некоторые дополнительные пользователи пользователей, которых необходимо создать после инициализации нового кластера
  users:
    admin:
      password: passadmin12356789
      options:
        - createrole
        - createdb

postgresql:
  listen: 0.0.0.0:5432
  connect_address: 192.168.1.23:5432
  data_dir: /var/lib/pgsql/14/data
  bin_dir: /usr/pgsql-14/bin
  config_dir: /var/lib/pgsql/14/data
  pgpass: /tmp/pgpass0
  authentication:
    replication:
      username: replicator
      password: rep-pass
    superuser:
      username: postgres
      password: postgres
    rewind:  # Не влияет на postgres 10 и ниже
      username: rewind_user
      password: rewind_password
  parameters:

    #------------------------------------------------------------------------------
    # CONNECTIONS AND AUTHENTICATION
    #------------------------------------------------------------------------------

    listen_addresses: '*'
    port: 5432

    #------------------------------------------------------------------------------
    # RESOURCE USAGE (except WAL)
    #------------------------------------------------------------------------------

    shared_buffers: 2048MB
    temp_buffers: 8MB
    work_mem: 4MB
    maintenance_work_mem: 512MB
    max_connections: 100
    dynamic_shared_memory_type: posix
    seq_page_cost: 1
    random_page_cost: 4
    cpu_operator_cost: 0.0025

    #------------------------------------------------------------------------------
    # REPORTING AND LOGGING
    #------------------------------------------------------------------------------

    # - Where to Log -
    log_destination: 'stderr'
    logging_collector: on
    log_timezone: 'Europe/Moscow'

    # - Locale and Formatting -

    datestyle: 'iso, dmy'
    timezone: 'Europe/Moscow'

    # - Kernel Resources -
    max_files_per_process: 10000
    commit_delay: 0

    # - Genetic Query Optimizer -
    from_collapse_limit: 8
    join_collapse_limit: 8

    #------------------------------------------------------------------------------
    # AUTOVACUUM
    #------------------------------------------------------------------------------

    autovacuum_max_workers: 4 
    vacuum_cost_limit: 200
    autovacuum_naptime: 10s
    autovacuum_vacuum_scale_factor: 0.01
    autovacuum_analyze_scale_factor: 0.003

    #------------------------------------------------------------------------------
    # LOCK MANAGEMENT
    #------------------------------------------------------------------------------

    max_locks_per_transaction: 64

    # - Shared Library Preloading -

    shared_preload_libraries: 'pg_stat_statements, pg_stat_kcache, pg_wait_sampling'

    #------------------------------------------------------------------------------
    # QUERY TUNING
    #------------------------------------------------------------------------------

    effective_cache_size: 4GB

    # - Checkpoints -

    checkpoint_completion_target: 0.9
    min_wal_size: 2000MB
    max_wal_size: 4000MB

    #------------------------------------------------------------------------------
    # WRITE-AHEAD LOG
    #------------------------------------------------------------------------------

    wal_buffers: 16MB

    # - Other Planner Options -

    default_statistics_target: 500

    # - Asynchronous Behavior -

    effective_io_concurrency: 200

    # - Planner Cost Constants -
    random_page_cost: 1.1

    # - Asynchronous Behavior -

    max_worker_processes: 8
    max_parallel_workers_per_gather: 2
    max_parallel_workers: 8
    max_parallel_maintenance_workers: 2

tags:
    nofailover: false
    noloadbalance: false
    clonefrom: false
    nosync: false



!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!ОБЯЗАТЕЛЬНО ЭТО КОНФИГУРАЦИЯ СДЕЛАНА ДЛЯ ТРЕЕХ НОД КЛАСТЕРА ETCD КЛАСТЕР!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!ДЛЯ PATRONI NODE3 IP 192.168.1.24 !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!ETCD CLUSTER 192.168.1.14:2379 192.168.1.15:2379 192.168.1.16:2379!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
Create the patroni.yml configuration file.
su postgres
vim /etc/patroni/patroni.yml
    /etc/patroni/patroni.yml
	
ОБЯЗАТЕЛЬНО СНОВА ВЫПОЛНИТЬ ЭТУ КОМАНДУ ПОСЛЕ СОЗДАНИЯ КОНФИГА ПАТРОНИ!!!
sudo chown -R  postgres:postgres /etc/patroni/
sudo chown -R  postgres:postgres /etc/patroni/
sudo chown -R  postgres:postgres /etc/patroni/


tail -f /var/log/messages
tail -f /var/log/messages
tail -f /var/log/messages

tail -f /var/lib/pgsql/14/data/log/postgresql-Thu.log
tail -f /var/lib/pgsql/14/data/log/postgresql-Thu.log
tail -f /var/lib/pgsql/14/data/log/postgresql-Thu.log

scope: mycluster
namespace: /pg_cluster/
name: pg_node3

restapi:
  listen: 0.0.0.0:8008
  connect_address: 192.168.1.24:8008


etcd:
  #Provide host to do the initial discovery of the cluster topology:
  hosts:
   - 192.168.1.14:2379
   - 192.168.1.15:2379
   - 192.168.1.16:2379



bootstrap:
  # этот раздел будет записан в Etcd:/<namespace>/<scope>/config после инициализации нового кластера
  # and all other cluster members will use it as a `global configuration` (и все остальные члены кластера будут использовать его как «глобальную конфигурацию».)
  dcs:
    ttl: 30
    loop_wait: 10
    retry_timeout: 10
    maximum_lag_on_failover: 1048576
    master_start_timeout: 10
    postgresql:
      use_pg_rewind: true
      use_slots: true
      parameters:
         wal_level: replica
         hot_standby: "on"
         wal_keep_segments: 8
         max_wal_senders: 5
         max_replication_slots: 5
         checkpoint_timeout: 30
#        wal_level: hot_standby
#        hot_standby: "on"
#        max_connections: 100
#        max_worker_processes: 8
#        wal_keep_segments: 8
#        max_wal_senders: 10
#        max_replication_slots: 10
#        max_prepared_transactions: 0
#        max_locks_per_transaction: 64
#        wal_log_hints: "on"
#        track_commit_timestamp: "off"
#        archive_mode: "on"
#        archive_timeout: 1800s
#        archive_command: mkdir -p ../wal_archive && test ! -f ../wal_archive/%f && cp %p ../wal_archive/%f
#        recovery_conf:
#        restore_command: cp ../wal_archive/%f %p

  # некоторые желаемые параметры для 'initdb'
  initdb:  # Примечание: это должен быть список (некоторые параметры требуют значений, другие являются переключателями)
  - encoding: UTF8
  # - data-checksums

  pg_hba:  # Добавьте следующие строки в pg_hba.conf после запуска 'initdb'
  - host replication replicator 127.0.0.1/32    md5
  - host replication replicator 192.168.1.22/32 md5  
  - host replication replicator 192.168.1.23/32 md5 
  - host replication replicator 192.168.1.24/32 md5
  - host all         rewind_user 192.168.1.22/32 md5
  - host all         rewind_user 192.168.1.23/32 md5
  - host all         rewind_user 192.168.1.24/32 md5
  - host replication all        127.0.0.1/32    md5

  # Некоторые дополнительные пользователи пользователей, которых необходимо создать после инициализации нового кластера
  users:
    admin:
      password: passadmin12356789
      options:
        - createrole
        - createdb

postgresql:
  listen: 0.0.0.0:5432
  connect_address: 192.168.1.24:5432
  data_dir: /var/lib/pgsql/14/data
  bin_dir: /usr/pgsql-14/bin
  config_dir: /var/lib/pgsql/14/data
  pgpass: /tmp/pgpass0
  authentication:
    replication:
      username: replicator
      password: rep-pass
    superuser:
      username: postgres
      password: postgres
    rewind:  # Не влияет на postgres 10 и ниже
      username: rewind_user
      password: rewind_password
  parameters:

    #------------------------------------------------------------------------------
    # CONNECTIONS AND AUTHENTICATION
    #------------------------------------------------------------------------------

    listen_addresses: '*'
    port: 5432

    #------------------------------------------------------------------------------
    # RESOURCE USAGE (except WAL)
    #------------------------------------------------------------------------------

    shared_buffers: 2048MB
    temp_buffers: 8MB
    work_mem: 4MB
    maintenance_work_mem: 512MB
    max_connections: 100
    dynamic_shared_memory_type: posix
    seq_page_cost: 1
    random_page_cost: 4
    cpu_operator_cost: 0.0025

    #------------------------------------------------------------------------------
    # REPORTING AND LOGGING
    #------------------------------------------------------------------------------

    # - Where to Log -
    log_destination: 'stderr'
    logging_collector: on
    log_timezone: 'Europe/Moscow'

    # - Locale and Formatting -

    datestyle: 'iso, dmy'
    timezone: 'Europe/Moscow'

    # - Kernel Resources -
    max_files_per_process: 10000
    commit_delay: 0

    # - Genetic Query Optimizer -
    from_collapse_limit: 8
    join_collapse_limit: 8

    #------------------------------------------------------------------------------
    # AUTOVACUUM
    #------------------------------------------------------------------------------

    autovacuum_max_workers: 4 
    vacuum_cost_limit: 200
    autovacuum_naptime: 10s
    autovacuum_vacuum_scale_factor: 0.01
    autovacuum_analyze_scale_factor: 0.003

    #------------------------------------------------------------------------------
    # LOCK MANAGEMENT
    #------------------------------------------------------------------------------

    max_locks_per_transaction: 64

    # - Shared Library Preloading -

    shared_preload_libraries: 'pg_stat_statements, pg_stat_kcache, pg_wait_sampling'

    #------------------------------------------------------------------------------
    # QUERY TUNING
    #------------------------------------------------------------------------------

    effective_cache_size: 4GB

    # - Checkpoints -

    checkpoint_completion_target: 0.9
    min_wal_size: 2000MB
    max_wal_size: 4000MB

    #------------------------------------------------------------------------------
    # WRITE-AHEAD LOG
    #------------------------------------------------------------------------------

    wal_buffers: 16MB

    # - Other Planner Options -

    default_statistics_target: 500

    # - Asynchronous Behavior -

    effective_io_concurrency: 200

    # - Planner Cost Constants -
    random_page_cost: 1.1

    # - Asynchronous Behavior -

    max_worker_processes: 8
    max_parallel_workers_per_gather: 2
    max_parallel_workers: 8
    max_parallel_maintenance_workers: 2

tags:
    nofailover: false
    noloadbalance: false
    clonefrom: false
    nosync: false



!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!после смотрим на лидере будут созданы слоты репликации для каждой ноды !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!через эти слоты реплика получает данные от мастера!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
	
postgres=# SELECT * FROM pg_replication_slots \gx
-[ RECORD 1 ]-------+----------
slot_name           | pg_node2
plugin              |
slot_type           | physical
datoid              |
database            |
temporary           | f
active              | t
active_pid          | 29072
xmin                |
catalog_xmin        |
restart_lsn         | 0/5000148
confirmed_flush_lsn |
wal_status          | reserved
safe_wal_size       |
two_phase           | f
-[ RECORD 2 ]-------+----------
slot_name           | pg_node3
plugin              |
slot_type           | physical
datoid              |
database            |
temporary           | f
active              | t
active_pid          | 29259
xmin                |
catalog_xmin        |
restart_lsn         | 0/5000148
confirmed_flush_lsn |
wal_status          | reserved
safe_wal_size       |
two_phase           | f



</pre>


# 3)PgBouncer

<pre> 

установка настройка PgBouncer
==========================================================================================================================================================================
==========================================================================================================================================================================
==========================================================================================================================================================================
==========================================================================================================================================================================
==========================================================================================================================================================================

ПРЕДИСЛОВИЕ:
------------------------------------------------------------------------------------------------------------------------------------
идем по принцепу:
Приложение > HAproxy > Pgbouncer > PostgreSQL (патроны)

HAProxy в этом случае будет выступать не только как прокси, 
но и как балансировщик нагрузки. Мы можем разделить соединение для чтения и записи на отдельные порты.

Вам нужен первый вариант, потому что pgBouncer — это пул соединений для PostgreSQL. 
Он должен быть близок к СУБД и быть один на один подключен каждый к своему экземпляру Postgresql. 
Что касается HAProxy, это балансировщик нагрузки, у вас может быть номер HAProxy, 
отличный от количества экземпляров Postgre\Patroni\pgBouncer. Кроме того, 
это даст вам возможность разделять запросы на чтение и запись в разные порты\базы данных.
------------------------------------------------------------------------------------------------------------------------------------


1)
что я делаю устанавливаю Pgbouncer на каждой машине с patroni на серверах:
192.168.1.22 local_otus_PARONI_vm1_centos8
192.168.1.23 local_otus_PARONI_vm2_centos8
192.168.1.24 local_otus_PARONI_vm3_centos8


sudo dnf install --disablerepo='*' *.rpm
sudo dnf install --disablerepo='*' *.rpm
sudo dnf install --disablerepo='*' *.rpm

Package chkconfig-1.19.1-1.el8.x86_64 is already installed.
Package platform-python-pip-9.0.3-20.el8.noarch is already installed.
Package platform-python-setuptools-39.2.0-6.el8.noarch is already installed.
Package python36-3.6.8-38.module_el8.5.0+895+a459eca8.x86_64 is already installed.
Package python3-pip-9.0.3-20.el8.noarch is already installed.
Package python3-psycopg2-2.8.6-1.rhel8.x86_64 is already installed.
Package python3-setuptools-39.2.0-6.el8.noarch is already installed.
Dependencies resolved.
===============================================================================================================================================================================================================
 Package                                        Architecture                                Version                                                    Repository                                         Size
===============================================================================================================================================================================================================
Installing:
 pgbouncer                                      x86_64                                      1.18.0-11.rhel8                                            @commandline                                      234 k
Upgrading:
 libpq5                                         x86_64                                      15.2-42.1PGDG.rhel8                                        @commandline                                      216 k

Transaction Summary
===============================================================================================================================================================================================================
Install  1 Package
Upgrade  1 Package

Total size: 450 k
Is this ok [y/N]: y


пред этими действиями в самом postgres мастере создаю пользователя 
create user vorori with password 'mypassword';
alter user vorori with superuser;


create user integrator with password 'mypassword';
create database test with owner integrator;


2)
чтобы в ручную не настраивать подготовленые конфгурационные файлы копирую в 
/etc/pgbouncer/

мои заметки:
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
auth_type = md5
auth_file = /etc/pgbouncer/userlist.txt
------------------------------------------------------------------------------------------------------------------------------------------------



3)
вставляем пароль пользователей (опять же скопировал уже в предыдушем шаге)
vim /etc/pgbouncer/userlist.txt
"vorori" "SCRAM-SHA-256$4096:wbQwgyduBNPbx76HdJw9uA==$pezbrqcrBDnEd/k3mITBhR4lSszWI+V8h0bFedDE5M4=:twBbKOlb80nh5Po+whwZ4RWFnMYFsNjMiL2FqG2dcXI="
"integrator" "SCRAM-SHA-256$4096:wbQwgyduBNPbx76HdJw9uA==$pezbrqcrBDnEd/k3mITBhR4lSszWI+V8h0bFedDE5M4=:twBbKOlb80nh5Po+whwZ4RWFnMYFsNjMiL2FqG2dcXI="



4)
--------
прегружаем службу балансирорвшика убеждаемся что все ок добавляем в автозагрузку

sudo systemctl enable --now pgbouncer
sudo service pgbouncer restart
sudo service pgbouncer status
sudo service pgbouncer stop
sudo service pgbouncer start
--------


5)
проверяем работу
--------
если надо зайти в админку pgbouncer 

psql -p 6432 -U vorori pgbouncer -h localhost
psql -p 6432 -U vorori pgbouncer -h localhost
psql -p 6432 -U vorori pgbouncer -h localhost
--------

--------
проверяем соединение с нашей тестовой базой

psql -U integrator -p 6432 -h localhost test
--------


--------
перечитывание конфигурации без перезапуска сервера:
RELOAD;

Информацию о других доступных командах можно посмотреть так:
SHOW HELP;
show LISTS;
show CLIENTS;
--------

--------
посмотреть какой порт слушает pgbouncer

netstat -lntp | grep pgbouncer
netstat -lntp | grep pgbouncer
netstat -lntp | grep pgbouncer
--------

--------
посмотреть какие клиенты подключены к pgbouncer

netstat -antp | grep 6432
netstat -antp | grep 6432
netstat -antp | grep 6432
--------

-------------------------------------------------------------------------------------------------------
https://eax.me/pgbouncer/
https://eax.me/pgbouncer/
https://eax.me/pgbouncer/

Проверяем, что все работает, запустив pgbench с 290 соединенинями:

/usr/pgsql-14/bin/pgbench -i test -U vorori -p 5432 -h localhost
/usr/pgsql-14/bin/pgbench -p 6432 -c 90 -C -T 60 -P 1 -U vorori -h 127.0.0.1 -d test
/usr/pgsql-14/bin/pgbench -p 6432 -h localhost -U vorori -d test -c 290 -C -T 60 -P 1
Если все было сделано правильно, pgbench будет превосходно работать, а команда SHOW CLIENTS;, 
выполненная в админке PgBouncer, покажет 290 соединений. 
При этом вывод ps wuax | grep postgres покажет, что реально запущено лишь 20 бэкендов PostgreSQL.

ps wuax | grep postgres
ps wuax | grep postgres
ps wuax | grep postgres
-------------------------------------------------------------------------------------------------------

</pre>

# 4) HAProxy

<pre> 

создал настроил две VM для HAPROXY соединяем их KeepAlived
local_local_otus_HAPROXY_vm4_haproxy_192.168.1.17
и
local_local_otus_HAPROXY_vm5_haproxy_192.168.1.25

6)
Настроить HAProxy
--------------------------------------------------------------------------
снова отключаем SELINUX
vim /etc/selinux/config
Измените SELINUX=enforcing на SELINUX=disabled

SELINUX=disabled


Перезагрузите все узлы, чтобы изменения selinux вступили в силу:
 
sudo shutdown -r now
--------------------------------------------------------------------------



7)
Настроить HAProxy
Отредактируйте файл haproxy.cfg на вашем первом узле ( патрони1 ) в нашем случае:
 
sudo mv /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg.orig


vim /etc/haproxy/haproxy.cfg
Добавьте следующую конфигурацию:
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
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
    server patroni1 192.168.1.22:6432 maxconn 300 check port 8008
    server patroni2 192.168.1.23:6432 maxconn 300 check port 8008
    server patroni3 192.168.1.24:6432 maxconn 300 check port 8008

listen standby_postgres_read
    bind *:5001
    balance leastconn
    option httpchk OPTIONS /replica
    http-check expect status 200
    default-server inter 3s fall 3 rise 2 on-marked-down shutdown-sessions
    server patroni1 192.168.1.22:6432 maxconn 300 check port 8008
    server patroni2 192.168.1.23:6432 maxconn 300 check port 8008
    server patroni3 192.168.1.24:6432 maxconn 300 check port 8008
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
	
	

8)
запускаем службу добавляем сервис в автозагрузку
systemctl stop haproxy
systemctl start haproxy
systemctl status haproxy
systemctl enable haproxy


9)
проверяем работу сервиса
http://192.168.1.17:7000/

обрашаем внимание на то что если запросы на запись то подключаемся по поррту 5000
если нужны запросы на чтение то порт 5001

В конфигурации haproxy есть два важных раздела прослушивания :
первичный , использующий порт 5000 для запросов на чтение/запись.
в режиме ожидания с использованием порта 5001 только для запросов на чтение.
Все три узла включены в оба раздела: это потому, что все серверы баз 
данных являются потенциальными кандидатами на роль первичных или вторичных. 
Patroni предоставляет встроенную поддержку REST API для мониторинга проверки работоспособности, 
которая отлично работает с HAProxy. HAProxy отправит HTTP-запрос на порт 8008 патрони, чтобы узнать, какую роль в настоящее время имеет каждый узел.
 
Конфигурация haproxy останется неизменной на каждом узле, поэтому убедитесь, 
что вы повторяете ту же конфигурацию на оставшихся узлах, прежде чем переходить к следующему.
	

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
еше один пример заметка:
При обращении на урл server:8008 (8008 это порт по умолчанию) Patroni возвращает отчет по состоянию кластера в json, 
а также отражает кодом ответа http является ли данный сервер мастером. Если является — будет ответ с кодом 200.
Если же нет, ответ с кодом 503.

Очень советую обратится в документацию на Patroni, http интерфейс там достаточно интересный, 
позволяется также принудительно переключать роли, и управлять кластером.
Аналогично, это можно делать при помощи консольной утилиты patronyctl.py, из поставки Patroni.

В соответствии с этой конфигурацией haproxy слушает порт 5000, и отправляет трафик с него на мастер сервер.

Проверка статуса происходит с интервалом в 1 секунду, для перевода сервера в даун требуется 3 неудачных ответа (код 500), 
для переключения сервера назад — 2 удачных ответа (с кодом 200).
В любой момент времени можно обратиться непосредственно на любой haproxy, и он корректно запроксирует трафик на мастер сервер.

Конфигурация haproxy достаточно простая:
global
maxconn 800

defaults
log global
mode tcp
retries 2
timeout client 30m
timeout connect 4s
timeout server 30m
timeout check 5s

frontend ft_postgresql
bind *:5000
default_backend postgres-patroni

backend postgres-patroni
  option httpchk

  http-check expect status 200
  default-server inter 3s fall 3 rise 2

  server {{ patroni_node_name }} {{ patroni_node_name }}.local:5432 maxconn 300 check port 8008
  server {{ patroni_node_name }} {{ patroni_node_name }}.local:5432 maxconn 300 check port 8008
  server {{ patroni_node_name }} {{ patroni_node_name }}.local:5432 maxconn 300 check port 8008
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
	
	




мои заметки чтобы было под рукой:
на просторах интернета нашел описание портов 
http://msutic.blogspot.com/2021/11/deploying-postgresql-140-for-high.html

Порты, необходимые для работы патрони/etcd/haproxy/postgresql, следующие:

5432 — стандартный порт PostgreSQL, используемый не самим PostgreSQL, а HAProxy
5000 — порт прослушивания PostgreSQL, используемый HAproxy для маршрутизации подключений к базе данных для записи узла
5001 — PostgreSQL порт прослушивания, используемый HAproxy для маршрутизации соединений с базой данных на узлы чтения
2380 — порт URL-адресов одноранговых узлов etcd, необходимый для связи членов etcd
2379 — клиентский порт etcd, необходимый любому клиенту, включая патрони, для связи с etcd
8008- Патронный порт API, необходимый HAProxy для проверки состояния узлов
7000 - Порт HAProxy для предоставления статистики прокси-сервера


10)
#подключаемся с сервера хапрокси на наш кластер
psql -U vorori -p 5000 -h localhost postgres

postgres=# select pg_read_file('/etc/hostname') as hostname;
 hostname
----------
 srv01   +


#подключаемся на ноды которые только для чтения
psql -U vorori -p 5001 -h localhost postgres
postgres=# select pg_read_file('/etc/hostname') as hostname;
 hostname
----------
 srv03   +

	
#подключаемся на ноды которые только для чтения	
psql -U vorori -p 5001 -h localhost postgres
postgres=# select pg_read_file('/etc/hostname') as hostname;
 hostname
----------
 srv02   +

11)
Усе приехали!!!!

</pre>



# 5) keepalived

<pre> 
создал настроил две VM для HAPROXY соединяем их KeepAlived
local_local_otus_HAPROXY_vm4_haproxy_192.168.1.17
и
local_local_otus_HAPROXY_vm5_haproxy_192.168.1.25

12)
Настроить keepalived
--------------------------------------------------------------------------
снова отключаем SELINUX
vim /etc/selinux/config
Измените SELINUX=enforcing на SELINUX=disabled

SELINUX=disabled


Перезагрузите все узлы, чтобы изменения selinux вступили в силу:
 
sudo shutdown -r now
--------------------------------------------------------------------------



13)
Настроить keepalived на ноде otus_HAPROXY_vm4_haproxy_192.168.1.17

Плавающий IP адрес (или виртуальный, далее VIP) используется для обеспечения отказоустойчивости в кластерах.
Кластер конфигурируется таким образом, что плавающий IP присвоен только одному узлу
в каждый момент времени. Если узел с VIP стал недоступен, то VIP присваивается standby узлу.
Тогда standby узел посылает оповещение (gratuitous ARP), чтобы оповестить остальных участников сети
о смене MAC адреса интерфейса с присвоенным VIP.

Простыми словами, standby узел "забирает" IP адрес себе, и все запросы начинают идти ему.


Установим keepalived на оба узла:
yum install keepalived
yum install psmisc-22.20-17.el7.x86_64  # для скрипта killall -0 haproxy



Далее, нам нужно следующий параметр Linux Kernel, чтобы включить поддержку
плавающих IP. На обоих узлах:
echo "net.ipv4.ip_nonlocal_bind = 1" >> /etc/sysctl.conf
sysctl -p


Теперь займемся конфигурацией keepalived, поместим в файл назначим виртуальный ip 192.168.1.26/24 
vim /etc/keepalived/keepalived.conf


! Configuration File for keepalived
 
vrrp_script chk_haproxy {
  script "killall -0 haproxy" # check the haproxy process
  interval 2                  # every 2 seconds
  weight 2                    # add 2 points if OK
}
 
vrrp_instance VI_1 {
  interface eth0              # interface to monitor тут надо указать имя сушествующего сетевого интерфейса!!!!
  state MASTER                # MASTER on local_otus_HAPROXY_vm4_haproxy, BACKUP on local_otus_HAPROXY_vm5_haproxy
  virtual_router_id 51
  priority 101                # 101 on local_otus_HAPROXY_vm4_haproxy, 100 local_otus_HAPROXY_vm5_haproxy
 
  virtual_ipaddress {
     192.168.1.26/24          # virtual ip address
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
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:15:5d:03:ae:15 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.17/24 brd 192.168.1.255 scope global noprefixroute eth0
       valid_lft forever preferred_lft forever
    inet 192.168.1.26/24 scope global secondary eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::215:5dff:fe03:ae15/64 scope link
       valid_lft forever preferred_lft forever
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:cf:ea:a1:a0 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:cfff:feea:a1a0/64 scope link
       valid_lft forever preferred_lft forever
5: vethd46c0b5@if4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
    link/ether ee:70:8e:fc:ae:50 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::ec70:8eff:fefc:ae50/64 scope link
       valid_lft forever preferred_lft forever

видим     inet 192.168.1.26/24 scope global secondary eth0           значит наш виртуальный ip поднялся и все норм !!!!



15)
подключаемся на вторую ноду local_otus_HAPROXY_vm5_haproxy_192.168.1.25 и вставляем этот конфиг
vim /etc/keepalived/keepalived.conf


! Configuration File for keepalived
 
vrrp_script chk_haproxy {
  script "killall -0 haproxy" # check the haproxy process
  interval 2                  # every 2 seconds
  weight 2                    # add 2 points if OK
}
 
vrrp_instance VI_1 {
  interface eth0              # interface to monitor тут надо указать имя сушествующего сетевого интерфейса!!!!
  state BACKUP                # MASTER on local_otus_HAPROXY_vm4_haproxy, BACKUP on local_otus_HAPROXY_vm5_haproxy
  virtual_router_id 51
  priority 100                # 101 on local_otus_HAPROXY_vm4_haproxy, 100 local_otus_HAPROXY_vm5_haproxy
 
  virtual_ipaddress {
    192.168.1.26/24           # virtual ip address
  }
 
  track_script {
    chk_haproxy
  }
}



16)
Описание:
killall -0 haproxy проверяет запущен ли процесс HAProxy.
Узел local_otus_HAPROXY_vm4_haproxy имеет приоритет 101, поэтому он всегда будет "забирать" VIP себе, если жив.
Таким образом, VIP будет назначен local_otus_HAPROXY_vm5_haproxy, только если local_otus_HAPROXY_vm4_haproxy перестал отвечать.


17)
Проверка
Проверим, что VIP присвоился интерфейсу eth0. Выполним на local_otus_HAPROXY_vm4_haproxy:

# ip addr | grep "inet" | grep "eth0"
    inet 192.168.1.17/24 brd 192.168.1.255 scope global noprefixroute eth0
    inet 192.168.1.26/24 scope global secondary eth0



Убьем HAProxy на local_otus_HAPROXY_vm4_haproxy_192.168.1.17 и посмотрим, что будет. На local_otus_HAPROXY_vm4_haproxy:
# systemctl stop haproxy.service


на HAProxy на local_otus_HAPROXY_vm5_haproxy :
 ip addr | grep "inet" | grep "eth0"
    inet 192.168.1.25/24 brd 192.168.1.255 scope global noprefixroute eth0
    inet 192.168.1.26/24 scope global secondary eth0
	
все работает!



</pre>


# 6) настраиваем бэкап используя WAL-G 

<pre> 


#### 1.1) install WAL-G


-- sudo rm /usr/local/bin/wal-g
wget https://github.com/philyuchkoff/wal-g-centos7/releases/download/2.0.1/wal-g-pg && mkdir /usr/local/bin/wal-g && sudo mv wal-g-pg /usr/local/bin/wal-g
sudo ls -l /usr/local/bin/wal-g

sudo rm -rf /var/postgre_backup && sudo mkdir /var/postgre_backup && sudo chmod 777 /var/postgre_backup
chmod 645 /usr/local/bin/wal-g/wal-g-pg
chown postgres:postgres /var/postgre_backup



#### 1.2)Создаем файл конфигурации для wal-g


sudo su - postgres
vim ~/.walg.json


-- https://github.com/wal-g/wal-g/blob/master/docs/PostgreSQL.md
-- https://github.com/wal-g/wal-g/blob/master/docs/STORAGES.md

{
    "WALG_FILE_PREFIX": "/var/postgre_backup",
    "WALG_COMPRESSION_METHOD": "brotli",
    "WALG_DELTA_MAX_STEPS": "5",
    "PGDATA": "/var/lib/pgsql/14/data",
    "PGHOST": "/var/run/postgresql/.s.PGSQL.5432"
}



-- опция для дебага
--     "WALG_LOG_LEVEL": "DEVEL"

mkdir /var/lib/pgsql/14/data/log_wal_g
chown postgres:postgres /var/lib/pgsql/14/data/log_wal_g
chmod 700 /var/lib/pgsql/14/data/log_wal_g


добавляю настройки в config patroni
patronictl -c /etc/patroni/patroni.yml edit-config
patronictl -c /etc/patroni/patroni.yml edit-config
patronictl -c /etc/patroni/patroni.yml edit-config

----------------------------------------------------------------------------------------------------------------------------------------------
loop_wait: 10
master_start_timeout: 10
maximum_lag_on_failover: 1048576
postgresql:
  parameters:
    archive_command: /usr/local/bin/wal-g/wal-g-pg wal-push "%p" >> /var/lib/pgsql/14/data/log_wal_g/archive_command.log 2>&1
    archive_mode: true
    archive_timeout: 60
    checkpoint_timeout: 30
    hot_standby: 'on'
    log_line_prefix: '%m [%p] %d %u %h (transaction ID %x)'
    log_statement: none
    max_connections: 102
    max_replication_slots: 5
    max_wal_senders: 5
    track_activity_query_size: 16384
    wal_keep_segments: 8
    wal_level: replica
  use_pg_rewind: true
  use_slots: true
retry_timeout: 10
ttl: 30
----------------------------------------------------------------------------------------------------------------------------------------------


-----------------------------------------------------------------------------
пререстартовать кластер применил настройки :
patronictl -c /etc/patroni/patroni.yml restart mycluster
patronictl -c /etc/patroni/patroni.yml restart mycluster
-----------------------------------------------------------------------------


----------------------------------------------------------------------------------------------------------------------------------------------
[postgres@localhost wal-g-pg]$ /usr/local/bin/wal-g/wal-g-pg backup-push /var/lib/pgsql/14/data
INFO: 2023/05/26 16:44:21.628413 Couldn't find previous backup. Doing full backup.
INFO: 2023/05/26 16:44:21.632486 Calling pg_start_backup()
INFO: 2023/05/26 16:44:21.674857 Starting a new tar bundle
INFO: 2023/05/26 16:44:21.674880 Walking ...
INFO: 2023/05/26 16:44:21.677056 Starting part 1 ...
INFO: 2023/05/26 16:44:22.976268 Packing ...
INFO: 2023/05/26 16:44:22.976567 Finished writing part 1.
INFO: 2023/05/26 16:44:22.976585 Starting part 2 ...
INFO: 2023/05/26 16:44:22.976591 /global/pg_control
INFO: 2023/05/26 16:44:22.977066 Finished writing part 2.
INFO: 2023/05/26 16:44:22.977072 Calling pg_stop_backup()
INFO: 2023/05/26 16:44:23.989410 Starting part 3 ...
INFO: 2023/05/26 16:44:23.989450 backup_label
INFO: 2023/05/26 16:44:23.989455 tablespace_map
INFO: 2023/05/26 16:44:23.989695 Finished writing part 3.
INFO: 2023/05/26 16:44:23.993523 Wrote backup with name base_00000001000000000000000A
[postgres@localhost wal-g-pg]$ /usr/local/bin/wal-g/wal-g-pg backup-push /var/lib/pgsql/14/data
INFO: 2023/05/26 16:44:37.737061 LATEST backup is: 'base_00000001000000000000000A'
INFO: 2023/05/26 16:44:37.737191 Delta backup from base_00000001000000000000000A with LSN 0/A000028.
INFO: 2023/05/26 16:44:37.745889 Calling pg_start_backup()
INFO: 2023/05/26 16:44:37.851908 Delta backup enabled
INFO: 2023/05/26 16:44:37.851925 Starting a new tar bundle
INFO: 2023/05/26 16:44:37.851951 Walking ...
INFO: 2023/05/26 16:44:37.852073 Starting part 1 ...
INFO: 2023/05/26 16:44:37.864488 Packing ...
INFO: 2023/05/26 16:44:37.864569 Finished writing part 1.
INFO: 2023/05/26 16:44:37.864598 Starting part 2 ...
INFO: 2023/05/26 16:44:37.864605 /global/pg_control
INFO: 2023/05/26 16:44:37.864767 Finished writing part 2.
INFO: 2023/05/26 16:44:37.864771 Calling pg_stop_backup()
INFO: 2023/05/26 16:44:38.907474 Starting part 3 ...
INFO: 2023/05/26 16:44:38.907534 backup_label
INFO: 2023/05/26 16:44:38.907539 tablespace_map
INFO: 2023/05/26 16:44:38.907738 Finished writing part 3.
INFO: 2023/05/26 16:44:38.911426 Wrote backup with name base_00000001000000000000000C_D_00000001000000000000000A

----------------------------------------------------------------------------------------------------------------------------------------------
проверяем что нет ошибок и все ок
cat /var/lib/pgsql/14/data/log/postgresql-Fri.log

2023-05-26 16:43:10.511 MSK [9771]    (transaction ID 0)LOG:  listening on IPv4 address "0.0.0.0", port 5432
2023-05-26 16:43:10.513 MSK [9771]    (transaction ID 0)LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
2023-05-26 16:43:10.519 MSK [9771]    (transaction ID 0)LOG:  listening on Unix socket "/tmp/.s.PGSQL.5432"
2023-05-26 16:43:10.524 MSK [9774]    (transaction ID 0)LOG:  database system was shut down at 2023-05-26 16:43:10 MSK
2023-05-26 16:43:10.528 MSK [9771]    (transaction ID 0)LOG:  database system is ready to accept connections
2023-05-26 16:43:10.529 MSK [9781]    (transaction ID 0)LOG:  pg_wait_sampling collector started

cat /var/lib/pgsql/14/data/log_wal_g/archive_command.log
INFO: 2023/05/26 16:44:10.775069 FILE PATH: 000000010000000000000009.br
INFO: 2023/05/26 16:44:21.923794 FILE PATH: 000000010000000000000001.br
INFO: 2023/05/26 16:44:23.063961 FILE PATH: 00000001000000000000000A.00000028.backup.br
INFO: 2023/05/26 16:44:23.080672 FILE PATH: 00000001000000000000000A.br
INFO: 2023/05/26 16:44:37.812206 FILE PATH: 00000001000000000000000B.br
INFO: 2023/05/26 16:44:37.969119 FILE PATH: 000000010000000000000002.br
INFO: 2023/05/26 16:44:37.983516 FILE PATH: 00000001000000000000000C.00000028.backup.br
INFO: 2023/05/26 16:44:37.990439 FILE PATH: 00000001000000000000000C.br
INFO: 2023/05/26 16:44:38.011426 FILE PATH: 000000010000000000000003.br
INFO: 2023/05/26 16:45:07.947696 FILE PATH: 000000010000000000000004.br
INFO: 2023/05/26 16:45:38.052529 FILE PATH: 00000001000000000000000D.br
----------------------------------------------------------------------------------------------------------------------------------------------


----------------------------------------------------------------------------------------------------------------------------------------------
смотрим бекапы в наличии:
[postgres@localhost wal-g-pg]$ /usr/local/bin/wal-g/wal-g-pg backup-list
name                                                     modified                  wal_segment_backup_start
base_00000001000000000000000A                            2023-05-26T16:44:23+03:00 00000001000000000000000A
base_00000001000000000000000C_D_00000001000000000000000A 2023-05-26T16:44:38+03:00 00000001000000000000000C
----------------------------------------------------------------------------------------------------------------------------------------------


### все настроенно можно добавлять задания в крон на репликах и будет счастье.

</pre>


# 7) Итоги

<pre> 
Создал отказоустойчивый кластер 3VM ETCD + 3VM patroni postgresql14 на этих же VM Pgbouncer 3штуки + 2VM HAproxy+keepalived = 8VM
Отказоустойчивость тестировал и в хвост и в гриву давал нагрузку через pgbench плюс виртуалки собрал на своем пк железо позволяет и теперь можно будет ставить эксперементы
по обновлению версии на 15 postgre и прикручу мониторинг Prometheus  и будет счастье!

</pre>
