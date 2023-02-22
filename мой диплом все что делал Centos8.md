
<pre>
Отключить Selinux (необязательно)
Если вы знакомы с SELINUX, вам следует настроить его соответствующим образом. 
Если у вас нет большого опыта, рекомендуется отключить selinux, чтобы избежать каких-либо трудностей при настройке кластера.
 
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


мои заметки:
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
</pre>



<pre>
------------------------------------------------------------------------------------
https://itisgood.ru/2019/06/18/kak-izmenit-imja-hosta-v-centos-7-8-i-fedora-30-29-28/
Прежде чем устанавливать имя хоста, сначала проверьте существующее имя хоста.

----------------
именуем каждую ноду
cat /etc/hosts

#на сервере 192.168.1.18
srv01

#на сервере 192.168.1.19
srv02

#на сервере 192.168.1.20
srv03
----------------


----------------
hostname -s
hostname -f
hostnamectl
----------------

----------------
hostname -s
localhost
----------------

----------------
hostname -f
localhost

-s, –short – используется для вывода короткого имени хоста
-f, –fqdn, –long – используется для вывода полного имени хоста (FQDN)
----------------

----------------
hostnamectl
   Static hostname: localhost.localdomain
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 93a73a8c198e4e17b1f8853d35cfb6a7
           Boot ID: 07afc630f43149b8811376b5272c6389
    Virtualization: microsoft
  Operating System: CentOS Linux 8 (Core)
       CPE OS Name: cpe:/o:centos:centos:8
            Kernel: Linux 4.18.0-147.3.1.el8_1.x86_64
      Architecture: x86-64


Static hostname – хранится в /etc/hostname для использования во время выполнения.

Transient hostname  – это динамическое имя хоста, управляемое ядром, которое может быть изменено сервером DHCP или mDNS во время выполнения. Используйте флаг –transient, чтобы установить временное имя хоста.
----------------

1)
#Выполняю наименование своих тркх машинок 
Изменить имя хоста с помощью hostnamectl
Чтобы установить постоянное имя хоста с помощью команды hostnamectl, используйте команду.
sudo hostnamectl set-hostname srv01 --static
sudo hostnamectl set-hostname srv02 --static
sudo hostnamectl set-hostname srv03 --static

1.1)
#в моем случае днс сервера нет поэтому я делаю это вручную
sudo hostnamectl set-hostname srv01 --transient
sudo hostnamectl set-hostname srv02 --transient
sudo hostnamectl set-hostname srv03 --transient

2)
Подтвердите свое новое имя хоста.
hostnamectl 

3)
Этот параметр автоматически обновит файл /etc/hostname.
cat /etc/hostname 
</pre>



<pre>
часть 1 Hyper-V окружение
==========================================================================================================================================================================
==========================================================================================================================================================================
==========================================================================================================================================================================
==========================================================================================================================================================================
==========================================================================================================================================================================
1)
создал сеть остнастка диспетчер виртуальных комутаторов "VM and PC" для обединения рабочей станции с серверами 
сеть 192.168.1.0 маска 255.255.255.0


2)
создал 1 родительский диск установил настроил Centos7 


3)
создал 4 диска разностных
Разностный, или дифференциальный (англ. differencing) виртуальный жесткий диск (VHD) 
является одним из трех типов виртуальных дисков, которые используются в Hyper-V. 
Разностный диск еще иногда называют «дочерним», так как он не является самостоятельным 
диском и в процессе работы полностью зависит от «родительского» диска. 


4)
создал 4vm
ТЕСТ_local_otus_diplom_vm1_centos8
ТЕСТ_local_otus_diplom_vm2_centos8
ТЕСТ_local_otus_diplom_vm3_centos8


5)
Запускаем каждый сервер и делаем настройки
настройка сеть на каждом сервере linux 


vim /etc/sysconfig/network-scripts/ifcfg-eth0
systemctl restart network


6) по итогу получили список серверов
ТЕСТ_local_otus_diplom_vm1_centos8          192.168.1.18
ТЕСТ_local_otus_diplom_vm2_centos8          192.168.1.19
ТЕСТ_local_otus_diplom_vm3_centos8          192.168.1.20


Имеется: 
ВСЕГО: 5 виртуальных машин

3 виртуальных машин на "Centos8" они же идут под базы данных Postgresql 14.6 + служба "Patroni" + "etcd" + "Pgbouncer" (Вреальном боевом контуре кластер "etcd" вынес бы на отдельные от СУБД сервера)
1 машина на "Centos7" "HAproxy" + "Docker" + "pgwatch2 + Grafana"


На всех 3 машинах  "local_otus_diplom_vm1_centos8", "local_otus_diplom_vm2_centos8", "local_otus_diplom_vm3_centos8" 
будут установлены сервисы "etcd" посредством которого, узлы будут введены в один общий кластер, 
"etcd" будет использоваться службой "Patroni" в целях хранения своей конфигурации,
каждый конфигурационный файл "patroni.yml", будет создан на каждом узле хранения баз данных в ручную, с почти идентичным настройками.

Предварительно на серверах "local_otus_diplom_vm1_centos8", "local_otus_diplom_vm2_centos8", "local_otus_diplom_vm3_centos8" установлены системы управления баз данных "PostgreSQL" 
но не запущенные и без первичной инициализации баз данных,
так же служба "PostgreSQL" снята с автозапуска, "patroni" самостоятельно будет управлять а так же конфигурировать "PostgreSQL".

==========================================================================================================================================================================
</pre>



<pre>

часть 2  etcd кластер
==========================================================================================================================================================================
==========================================================================================================================================================================
==========================================================================================================================================================================
==========================================================================================================================================================================
==========================================================================================================================================================================



1)
Установка "etcd"

скачать https://storage.googleapis.com/etcd/v3.3.11/etcd-v3.3.11-linux-amd64.tar.gz

tar xvf etcd-v3.3.11-linux-amd64.tar.gz
cd etcd-v3.3.11-linux-amd64
mv etcd etcdctl /usr/local/bin

2)
проверяем версию
etcd --version
etcd --version

/usr/local/bin/etcd --version
/usr/local/bin/etcd --version

etcd Version: 3.3.11
Git SHA: 2cf9e51d2
Go Version: go1.10.7
Go OS/Arch: linux/amd64


документация:
https://etcd.io/docs/v3.3/
https://etcd.io/docs/v3.3/
https://etcd.io/docs/v3.3/





3)
Создаём окружение для работы сервиса "etcd":

mkdir -p /var/lib/etcd/
mkdir /etc/etcd
groupadd --system etcd
useradd -s /sbin/nologin --system -g etcd etcd
chown -R etcd:etcd /var/lib/etcd/


4)
Создаём для "etcd" как службы, служебный файл:

vim /etc/systemd/system/etcd.service

[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target
[Service]
Type=notify
WorkingDirectory=/var/lib/etcd/
EnvironmentFile=-/etc/etcd/etcd.conf
User=etcd
# set GOMAXPROCS to number of processors
ExecStart=/bin/bash -c "GOMAXPROCS=$(nproc) /usr/local/bin/etcd --name=\"${ETCD_NAME}\" --data-dir=\"${ETCD_DATA_DIR}\" --listen-client-urls=\"${ETCD_LISTEN_CLIENT_URLS}\""
Restart=on-failure
LimitNOFILE=65536
[Install]
WantedBy=multi-user.target

5)
Создаём конфигурацию начального/мастер узла кластера для всех трех нод меняю соответствующие значения:
vim /etc/etcd/etcd.conf


------------------------------------------------------------------------------------------------------
для первой ноды srv01
#[Member]
ETCD_DATA_DIR="/var/lib/etcd"
ETCD_LISTEN_PEER_URLS="http://0.0.0.0:2380"
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
ETCD_NAME="srv01"
ETCD_HEARTBEAT_INTERVAL="1000"
ETCD_ELECTION_TIMEOUT="5000"

#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.1.18:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.1.18:2379"
ETCD_INITIAL_CLUSTER="srv01=http://192.168.1.18:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd_cluster_777"
ETCD_INITIAL_CLUSTER_STATE="new"
------------------------------------------------------------------------------------------------------


------------------------------------------------------------------------------------------------------
для второй ноды первичная настройка srv02
#[Member]
ETCD_DATA_DIR="/var/lib/etcd"
ETCD_LISTEN_PEER_URLS="http://0.0.0.0:2380"
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
ETCD_NAME="srv02"
ETCD_HEARTBEAT_INTERVAL="1000"
ETCD_ELECTION_TIMEOUT="5000"

#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.1.19:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.1.19:2379"
ETCD_INITIAL_CLUSTER="srv02=http://192.168.1.19:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd_cluster_777"
ETCD_INITIAL_CLUSTER_STATE="new"
------------------------------------------------------------------------------------------------------


------------------------------------------------------------------------------------------------------
для третьей ноды первичная настройка srv03
#[Member]
ETCD_DATA_DIR="/var/lib/etcd"
ETCD_LISTEN_PEER_URLS="http://0.0.0.0:2380"
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
ETCD_NAME="srv03"
ETCD_HEARTBEAT_INTERVAL="1000"
ETCD_ELECTION_TIMEOUT="5000"

#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.1.20:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.1.20:2379"
ETCD_INITIAL_CLUSTER="srv02=http://192.168.1.20:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd_cluster_777"
ETCD_INITIAL_CLUSTER_STATE="new"
------------------------------------------------------------------------------------------------------


мои заметки:
------------------------------------------------------------------------------------------------------
Рассмотрим введённые параметры:
ETCD_DATA_DIR - указывает расположение каталога данных кластера

ETCD_LISTEN_PEER_URLS - задаёт схему и точку подключения для остальных узлов кластера, по шаблону scheme://IP:port. 
Схема может быть http, https. Альтернатива, unix:// или unixs:// для юникс сокетов. 
Если в качестве IP адреса указано 0.0.0.0, то указанный порт будет прослушиваться на всех интерфейсах.

ETCD_LISTEN_CLIENT_URLS - задаёт схему и точку подключения для клиентов кластера. В остальном совпадает с ETCD_LISTEN_PEER_URLS.

ETCD_NAME - человекочитаемое имя этого узла кластера. Должно быть уникально в кластере. 
Для первого узла может быть любым. Для последующих должно совпадать с именем, указанным при добавлении узла.

ETCD_HEARTBEAT_INTERVAL - время в миллисекудах, между рассылками лидером оповещений о том, что он всё ещё лидер. 
Рекомендуется задавать с учётом сетевой задержки между узлами кластера.

ETCD_ELECTION_TIMEOUT - время в миллисекундах, которое проходит между последним принятым оповещением 
от лидера кластера, до попытки захватить роль лидера на ведомом узле. Рекомендуется задавать его в несколько раз большим, чем ETCD_HEARTBEAT_INTERVAL. 
Более подробно о этих параметрах можно прочесть в документации.

ETCD_INITIAL_ADVERTISE_PEER_URLS - Список равноправных URL-адресов, по которым его могут найти остальные узлы кластера. 
Эти адреса используются для передачи данных по кластеру. По крайней мере, один из этих адресов должен быть маршрутизируемым для всех членов кластера. 
Могут содержать доменные имена. Используется только при первом запуске нового узла кластера.

ETCD_ADVERTISE_CLIENT_URLS - Список равноправных URL-адресов, по которым его могут найти остальные узлы кластера. 
Эти адреса используются для передачи данных по кластеру. По крайней мере, один из этих адресов должен 
быть маршрутизируемым для всех членов кластера. Могут содержать доменные имена.

ETCD_INITIAL_CLUSTER - Список узлов кластера на момент запуска. Используется только при первом запуске нового узла кластера.

ETCD_INITIAL_CLUSTER_TOKEN - Токен кластера. Должен совпадать на всех узлах кластера. Используется только при первом запуске нового узла кластера.

ETCD_INITIAL_CLUSTER_STATE - может принимать два значения "new" и "existing". Значение "new" используется 
при первом запуске первого узла в кластере. При значении "existing", узел при старте будет пытаться установить связь с остальными узлами кластера.
------------------------------------------------------------------------------------------------------




6)
Запускаем "etcd" службу и открываем порты:
systemctl daemon-reload
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
member 7aa04f7c10ace564 is healthy: got healthy result from http://192.168.1.18:2379
cluster is healthy



9)
Добавляем второй узел выполняем команну на добавление второго узала на первой ноде выполняю команду на присоединение!!!:
etcdctl member add srv02 http://192.168.1.19:2380
Added member named srv02 with ID fb05c721143a1777 to cluster

#бязательно на втором узле вставляем эти данные в конфиг etcd (только после этого стартуем службу на второй ноде чтобы проверить что все ок)
vim /etc/etcd/etcd.conf

ETCD_NAME="srv02"
ETCD_INITIAL_CLUSTER="srv01=http://192.168.1.18:2380,srv02=http://192.168.1.19:2380"
ETCD_INITIAL_CLUSTER_STATE="existing"



10)
Устанавливаем в аналогичном порядке "etcd" на втором узле, но изменяем конфигурацию под атрибуты данного узла:
vim /etc/etcd/etcd.conf

так выглядит обновленный конфигурационный файл на второй ноде
-------------------------------------------------------------------------------------------
#[Member]
ETCD_DATA_DIR="/var/lib/etcd"
ETCD_LISTEN_PEER_URLS="http://0.0.0.0:2380"
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
ETCD_NAME="srv02"
ETCD_HEARTBEAT_INTERVAL="1000"
ETCD_ELECTION_TIMEOUT="5000"

#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.1.19:2380"                                     # меняю на ip второго узла  "http://192.168.1.19:2380"   
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.1.19:2379"                                           # меняю на ip второго узла  "http://192.168.1.19:2379"  
ETCD_INITIAL_CLUSTER="srv01=http://192.168.1.18:2380,srv02=http://192.168.1.19:2380"            # сюда вставляю перечесляется кворум "srv01=http://192.168.1.18:2380,srv02=http://192.168.1.19:2380"   
ETCD_INITIAL_CLUSTER_TOKEN="etcd_cluster_777"                                                   # имя кластера остается прежнее "etcd_cluster_777" 
ETCD_INITIAL_CLUSTER_STATE="existing"                                                           #"Обратите внимание на флаг, он отличается так же"
-------------------------------------------------------------------------------------------


11)
Запускаем службу и проверяем состав кластера на состояние если все ок добавляю в автозагрузку:
systemctl enable etcd
systemctl start etcd
systemctl stop etcd
systemctl status etcd
etcdctl member list



12)
проверяю статус кластера
etcdctl cluster-health
member 54c645c18dbd32db is healthy: got healthy result from http://192.168.1.18:2379
member f13886e582053321 is healthy: got healthy result from http://192.168.1.19:2379
cluster is healthy



12.2)
добавляю третью ноду
на первой ноде выполняю команду на присоединение
etcdctl member add srv03 http://192.168.1.20:2380
Added member named srv03 with ID 69dad9f443ce9230 to cluster


#бязательно на втором узле вставляем эти данные в конфиг etcd (только после этого стартуем службу на второй ноде чтобы проверить что все ок)
vim /etc/etcd/etcd.conf
ETCD_NAME="srv03"
ETCD_INITIAL_CLUSTER="srv01=http://192.168.1.18:2380,srv03=http://192.168.1.20:2380,srv02=http://192.168.1.19:2380"
ETCD_INITIAL_CLUSTER_STATE="existing"




12.3)
меняю конфиг под третью ноду на третей ноде
vim /etc/etcd/etcd.conf

так выглядит обновленный конфигурационный файл на третьей ноде
-------------------------------------------------------------------------------------------
#[Member]
ETCD_DATA_DIR="/var/lib/etcd"
ETCD_LISTEN_PEER_URLS="http://0.0.0.0:2380"
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
ETCD_NAME="srv03"
ETCD_HEARTBEAT_INTERVAL="1000"
ETCD_ELECTION_TIMEOUT="5000"

#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.1.20:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.1.20:2379"
ETCD_INITIAL_CLUSTER="srv01=http://192.168.1.18:2380,srv03=http://192.168.1.20:2380,srv02=http://192.168.1.19:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd_cluster_777"
ETCD_INITIAL_CLUSTER_STATE="existing"
-------------------------------------------------------------------------------------------


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
ETCD_INITIAL_CLUSTER="srv01=http://192.168.1.18:2380,srv03=http://192.168.1.20:2380,srv02=http://192.168.1.19:2380"
ETCD_INITIAL_CLUSTER_STATE="new"



после того как приводим к едиообразию рестартуем каждую службу по очереди проверяем статус кластера ectd 
vim /etc/etcd/etcd.conf

etcdctl member list
etcdctl cluster-health
systemctl start etcd
systemctl stop etcd
systemctl status etcd



13.1)
ЕСЛИ ТРЕБУЕТСЯ ПРЕСОЗДАТЬ НОДУ!!!!!!!!!!!!!!!!!!!!
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

etcdctl member add srv01 http://192.168.1.18:2380
etcdctl member add srv01 http://192.168.1.18:2380


etcdctl -u root member add srv01 http://192.168.1.14:2380
etcdctl -u root member add srv01 http://192.168.1.14:2380

etcdctl member list
На проблемной ноде редактируем конфиг /etc/etcd/etcd.conf, прописывая в настройке
vim /etc/etcd/etcd.conf

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


hwclock --set --date "21 Feb 2023 10:37:30"

rm -rf /var/lib/etcd/member/snap/*
rm -rf /var/lib/etcd/member/wal/*


13.2)
Далее нам необходимо создать пользователя кластера с высокими правами, для внутренней смежной коммуникацией между "patroni" и кластером "etcd":
etcdctl user add root
New password: 12345678
User root created

etcdctl user get root
User: root
Roles:  root

---------------------------------------------------------------------
#вот этот пунк НЕ ВЫПОЛНЯЛ!!!!!!!!!!!!!!!!! аутентификацию не включал!!!! и соответственно в файле /etc/patroni/patroni.yml пароль от пользователя кластера etcd не прописывал
etcdctl auth enable
Authentication Enabled
---------------------------------------------------------------------

etcdctl --username root user get root
Password: 
User: root
Roles:  root
</pre>


<pre>
часть 3 Установка и настройка сервиса "patroni" на "Centos8"
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


вот полный список установки пакетов заранее скаченных с  донора который имеет доступ интернет (проделывам установку на всех трех нодах):

'2)etcd'
'3)postgresql14'
'4)glibc'
'5)libzstd'
'6)python3-devel'
'7)python3-rpm'
'8)patroni-etcd'
'9)config_for_patroni'


2)
#основные пакеты и зависемости
yum install --downloadonly --downloaddir=/tmp/postgresql14 postgresql14-contrib postgresql14-server pg_stat_kcache_14 pg_wait_sampling_14
yum install --downloadonly --downloaddir=/tmp/glibc glibc
yum install --downloadonly --downloaddir=/tmp/libzstd libzstd
yum install --downloadonly --downloaddir=/tmp/python3-devel gcc python3-devel
yum install --downloadonly --downloaddir=/tmp/python3-rpm python3-rpm
yum install --downloadonly --downloaddir=/tmp/patroni-etcd patroni-etcd

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


!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!ОБЯЗАТЕЛЬНО ЭТО КОНФИГУРАЦИЯ СДЕЛАНА ДЛЯ ОДНОЙ TCP НОДЫ КЛАСТЕРА ETCD!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
Для полноты картины, когда вы закончите, вы можете вернуться в предыдущий режим/режим по умолчанию с помощью:
:set nopaste
------------------------------------------------------------------------------------------------------------------------------------------------------
scope: mycluster
namespace: /pg_cluster/
name: pg_node1

restapi:
  listen: 0.0.0.0:8008
  connect_address: 192.168.1.18:8008


etcd:
  #Provide host to do the initial discovery of the cluster topology:
  host: 192.168.1.18:2379
  #hosts:
  #- host1:port1
  #- host2:port2



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
  - host replication replicator 192.168.1.18/32 md5  
  - host replication replicator 192.168.1.19/32 md5 
  - host replication replicator 192.168.1.20/32 md5
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
  connect_address: 192.168.1.18:5432
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

    shared_buffers: 128MB
    temp_buffers: 8MB
    work_mem: 4MB
    maintenance_work_mem: 64MB
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
    min_wal_size: 1GB
    max_wal_size: 2GB

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

------------------------------------------------------------------------------------------------------------------------------------------------------

мои заметки:
--------------------------------------
В приведённом примере настроек, есть ряд параметров, влияющих на выполнение переключения на резервный сервер.

maximum_lag_on_failover - максимальное количество байт, на которые может отставать резервный сервер от ведущего, участвующий в выборах нового лидера.

master_start_timeout - задержка в секундах, между обнаружением аварийной ситуации и началом отработки переключения на резервный сервер. 
По умолчанию 300 секунд. Если задано 0, то переключение начнётся немедленно. При использовании асинхронной репликации (как в приведённом примере) 
это может привести к потере последних транзакций. Максимальное время переключения на реплику 
равно "loop_wait" + "master_start_timeout" + "loop_wait". Если "master_start_timeout" установленно в 0, то это время становится равно значению параметра "loop_wait".

Подробнее о возможных вариантах репликации можно прочитать в документации
https://patroni.readthedocs.io/en/latest/replication_modes.html#replication-modes
--------------------------------------

!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!ОБЯЗАТЕЛЬНО ЭТО КОНФИГУРАЦИЯ СДЕЛАНА ДЛЯ ОДНОЙ НОДЫ КЛАСТЕРА ETCD!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!



6)
Создаём сервис для запуска службы "patroni":
служба уже создана тк. устанавливали из пакетов


7)
Запускаем службу "patroni" и открываем порты:
systemctl daemon-reload

sudo systemctl daemon-reload
sudo systemctl enable patroni.service
sudo systemctl start patroni.service
sudo systemctl start patroni.service
sudo systemctl status patroni.service
sudo systemctl stop patroni.service

systemctl start patroni.service

patronictl -c /etc/patroni/patroni.yml list
patronictl -c /etc/patroni/patroni.yml list
patronictl -c /etc/patroni/patroni.yml list

[root@localhost patroni]# patronictl -c /etc/patroni/patroni.yml list
+ Cluster: batman (7197425693358410092) -----+----+-----------+
| Member      | Host      | Role   | State   | TL | Lag in MB |
+-------------+-----------+--------+---------+----+-----------+
| postgresql0 | 127.0.0.1 | Leader | running |  4 |           |
+-------------+-----------+--------+---------+----+-----------+


после старта всех трех нод
[root@srv01 tmp]# patronictl -c /etc/patroni/patroni.yml list
+ Cluster: mycluster (7199603018565937232) ---+----+-----------+
| Member   | Host         | Role    | State   | TL | Lag in MB |
+----------+--------------+---------+---------+----+-----------+
| pg_node1 | 192.168.1.18 | Leader  | running |  1 |           |
| pg_node2 | 192.168.1.19 | Replica | running |  1 |         0 |
| pg_node3 | 192.168.1.20 | Replica | running |  1 |         0 |
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
</pre>









<pre>
часть 4 установка настройка HAProxy и PgBouncer
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


у меня HAProxy установлен:
на centos7 ip 192.168.1.17
local_otus_diplom_vm4_haproxy_centos7


1)
что я делаю устанавливаю Pgbouncer на каждой машине с patroni на серверах:
192.168.1.18 local_otus_diplom_vm1_centos8
192.168.1.19 local_otus_diplom_vm2_centos8
192.168.1.20 local_otus_diplom_vm3_centos8


sudo dnf install --disablerepo='*' *.rpm
sudo dnf install --disablerepo='*' *.rpm
sudo dnf install --disablerepo='*' *.rpm



пред этими действиями в самом postgres мастере создаю пользователя 
create user vorori with password 'mypassword';
alter user vorori with superuser;
create database test;


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

psql -U vorori -p 6432 -h localhost test
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
/usr/pgsql-14/bin/pgbench -p 6432 -c 290 -C -T 60 -P 1 -U vorori -h 127.0.0.1 -d test
/usr/pgsql-14/bin/pgbench -p 6432 -h localhost -U vorori -d test -c 290 -C -T 60 -P 1
Если все было сделано правильно, pgbench будет превосходно работать, а команда SHOW CLIENTS;, 
выполненная в админке PgBouncer, покажет 290 соединений. 
При этом вывод ps wuax | grep postgres покажет, что реально запущено лишь 20 бэкендов PostgreSQL.

ps wuax | grep postgres
ps wuax | grep postgres
ps wuax | grep postgres
-------------------------------------------------------------------------------------------------------



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
    server patroni1 192.168.1.18:6432 maxconn 300 check port 8008
    server patroni2 192.168.1.19:6432 maxconn 300 check port 8008
    server patroni3 192.168.1.20:6432 maxconn 300 check port 8008

listen standby_postgres_read
    bind *:5001
    balance leastconn
    option httpchk OPTIONS /replica
    http-check expect status 200
    default-server inter 3s fall 3 rise 2 on-marked-down shutdown-sessions
    server patroni1 192.168.1.18:6432 maxconn 300 check port 8008
    server patroni2 192.168.1.19:6432 maxconn 300 check port 8008
    server patroni3 192.168.1.20:6432 maxconn 300 check port 8008
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
	
	
	

8)
запускаем службу добавляем сервис в автозагрузку
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
	

-------------------------------------------------------------
еше один пример:
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
-------------------------------------------------------------
	
	




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
	
	
<pre>
часть 5 установка настройка docker и pgwatch2
==========================================================================================================================================================================
==========================================================================================================================================================================
==========================================================================================================================================================================
==========================================================================================================================================================================
==========================================================================================================================================================================


1)
устанавливаем docker	
yum install *.rpm

Dependencies Resolved

================================================================================================================================================================================================
 Package                                      Arch                      Version                                      Repository                                                            Size
================================================================================================================================================================================================
Installing:
 audit-libs-python                            x86_64                    2.8.5-4.el7                                  /audit-libs-python-2.8.5-4.el7.x86_64                                316 k
 checkpolicy                                  x86_64                    2.5-8.el7                                    /checkpolicy-2.5-8.el7.x86_64                                        1.2 M
 container-selinux                            noarch                    2:2.119.2-1.911c772.el7_8                    /container-selinux-2.119.2-1.911c772.el7_8.noarch                     41 k
 containerd.io                                x86_64                    1.4.12-3.1.el7                               /containerd.io-1.4.12-3.1.el7.x86_64                                 108 M
 docker-ce                                    x86_64                    3:20.10.12-3.el7                             /docker-ce-20.10.12-3.el7.x86_64                                      96 M
 docker-ce-cli                                x86_64                    1:20.10.12-3.el7                             /docker-ce-cli-20.10.12-3.el7.x86_64                                 144 M
 docker-ce-rootless-extras                    x86_64                    20.10.12-3.el7                               /docker-ce-rootless-extras-20.10.12-3.el7.x86_64                      20 M
 docker-scan-plugin                           x86_64                    0.12.0-3.el7                                 /docker-scan-plugin-0.12.0-3.el7.x86_64                               13 M
 fuse-overlayfs                               x86_64                    0.7.2-6.el7_8                                /fuse-overlayfs-0.7.2-6.el7_8.x86_64                                 116 k
 fuse3-libs                                   x86_64                    3.6.1-4.el7                                  /fuse3-libs-3.6.1-4.el7.x86_64                                       270 k
 libcgroup                                    x86_64                    0.41-21.el7                                  /libcgroup-0.41-21.el7.x86_64                                        134 k
 libseccomp                                   x86_64                    2.3.1-4.el7                                  /libseccomp-2.3.1-4.el7.x86_64                                       297 k
 libsemanage-python                           x86_64                    2.5-14.el7                                   /libsemanage-python-2.5-14.el7.x86_64                                441 k
 policycoreutils-python                       x86_64                    2.5-34.el7                                   /policycoreutils-python-2.5-34.el7.x86_64                            1.2 M
 python-IPy                                   noarch                    0.75-6.el7                                   /python-IPy-0.75-6.el7.noarch                                        119 k
 setools-libs                                 x86_64                    3.3.8-4.el7                                  /setools-libs-3.3.8-4.el7.x86_64                                     1.8 M
 slirp4netns                                  x86_64                    0.4.3-4.el7_8                                /slirp4netns-0.4.3-4.el7_8.x86_64                                    169 k


2)
стартуем службу добавляем в автозагрузку
sudo systemctl start docker
sudo systemctl status docker
sudo systemctl enable docker

3)
#устанавливаю pgwatch2
docker pull postgresai/pgwatch2

4)
#пренесли архив и закружаем в docker
а этой командой для выполнения загрузки ипользуется обратная операция docker load
docker load -i postgresai_pgwatch2.tar


5)
#после того как установили и запустили docker запускаем наш контерер pgwatch2

docker images | grep postg
postgresai/pgwatch2   latest    cbfeb0bb9182   3 weeks ago   1.17GB



sudo docker run -d --restart=unless-stopped --name pgwatch2 \
  -p 3000:3000 -p 8080:8080 -p 8081:8081 \
  -v /var/lib/pgwatch2/postgresql:/var/lib/postgresql \
  -v /var/lib/pgwatch2/influxdb:/var/lib/influxdb \
  -v grafana:/var/lib/grafana \
  -v pgwatch2:/pgwatch2/persistent-config \
  -e PW2_IRETENTIONDAYS=3 \
  -e PW2_WEBNOANONYMOUS=true \
  -e PW2_WEBUSER=admin \
  -e PW2_WEBPASSWORD=2338484 \
  -e PW2_GRAFANANOANONYMOUS=true \
  -e PW2_GRAFANAUSER=admin \
  -e PW2_GRAFANAPASSWORD=2338484 \
  postgresai/pgwatch2



проверяем что все работает
netstat -ltupn | grep docker
tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN      3278/docker-proxy
tcp        0      0 0.0.0.0:8081            0.0.0.0:*               LISTEN      3262/docker-proxy
tcp        0      0 0.0.0.0:3000            0.0.0.0:*               LISTEN      3294/docker-proxy
tcp6       0      0 :::8080                 :::*                    LISTEN      3282/docker-proxy
tcp6       0      0 :::8081                 :::*                    LISTEN      3266/docker-proxy
tcp6       0      0 :::3000                 :::*                    LISTEN      3298/docker-proxy


6)
#после через браузер открываем наши ссылки
#После запуска его можно открыть в браузере c паролями которыми запустили выше в команде:

Графана -- на порту 3000
Веб-интерфейс администратора — 8080
статус pgwatch (самопроверка) — 8081
Затем нам нужно перейти к: 8080 и добавить машину, которую нужно отслеживать.

http://192.168.1.17:3000/login
http://192.168.1.17:8080/login
http://192.168.1.17:8081/






7)
#создаем пользователя и добавляем права доступа 
#Во-первых, на хостах Postgres мы хотим наблюдать:
#Пользователь БД, который будет использоваться pgwatch2 + необходимые ему разрешения

CREATE ROLE pgwatch2 WITH LOGIN PASSWORD 'asfdsadfesadSADsdf1';
GRANT pg_monitor TO pgwatch2;
GRANT EXECUTE ON FUNCTION pg_stat_file(text) to pgwatch2;
GRANT EXECUTE ON FUNCTION pg_ls_dir(text) TO pgwatch2;
GRANT EXECUTE ON FUNCTION pg_wait_sampling_reset_profile() TO pgwatch2; --if pg_wait_sampling extension is used
GRANT CONNECT ON DATABASE test TO pgwatch2;
GRANT USAGE ON SCHEMA public TO pgwatch2;

#Добавить расширения
#Для большинства отслеживаемых баз данных чрезвычайно полезно (для устранения проблем с производительностью) такжеактивировать расширение pg_stat_statements:

CREATE EXTENSION pg_stat_statements;
CREATE EXTENSION pg_stat_kcache;
CREATE EXTENSION pg_wait_sampling;


8)
#Добавьте хост pgwatch2 в pg_hba.conf
vim /var/lib/pgsql/14/data/pg_hba.conf

host    all             pgwatch2        192.168.1.17/32         md5

9)
#применить настройки 
patronictl -c /etc/patroni/patroni.yml reload mycluster pg_node1
patronictl -c /etc/patroni/patroni.yml reload mycluster pg_node2
patronictl -c /etc/patroni/patroni.yml reload mycluster pg_node3

</pre>






<pre>
часть 6 восстановление потеряной ноды
==========================================================================================================================================================================
==========================================================================================================================================================================
==========================================================================================================================================================================
==========================================================================================================================================================================
==========================================================================================================================================================================

Create the patroni.yml configuration file.
su postgres
vim /etc/patroni/patroni.yml


меняю настройки wal для чистоты экспреримента


   min_wal_size: 128MB
   max_wal_size: 256MB


пречитываем настройки и делаем рестарт каждой ноды 
-----------------------------------
ЕСЛИ НАДО ПРЕЧИТАТЬ НАСТРОЙКИ patroni

patronictl -c /etc/patroni/patroni.yml reload mycluster pg_node1
patronictl -c /etc/patroni/patroni.yml restart mycluster pg_node1
patronictl -c /etc/patroni/patroni.yml reload mycluster pg_node2
patronictl -c /etc/patroni/patroni.yml restart mycluster pg_node2
patronictl -c /etc/patroni/patroni.yml reload mycluster pg_node3
patronictl -c /etc/patroni/patroni.yml restart mycluster pg_node3
-----------------------------------

подключаемся 
psql -U vorori -h localhost postgres

и

проверяем
show max_wal_size;

имитирую выключение по питанию на srv01



даю нагрузку на мастера через haproxy port 5000

root@localhost arh]# /usr/pgsql-14/bin/pgbench -c 290 -C -j 2 -P 10 -T 3600 -M extended test -p 5000 -U vorori -h localhost
Password:
pgbench (14.6)
starting vacuum...end.
progress: 10.0 s, 154.7 tps, lat 1595.840 ms stddev 264.530
progress: 20.0 s, 176.3 tps, lat 1684.238 ms stddev 191.383
progress: 30.0 s, 175.6 tps, lat 1647.168 ms stddev 136.893
progress: 40.0 s, 182.5 tps, lat 1560.239 ms stddev 38.552
progress: 50.0 s, 165.0 tps, lat 1777.687 ms stddev 256.559
progress: 60.0 s, 174.0 tps, lat 1663.561 ms stddev 173.050
progress: 70.0 s, 178.8 tps, lat 1573.921 ms stddev 89.377
progress: 80.0 s, 180.8 tps, lat 1637.445 ms stddev 153.989
progress: 90.0 s, 172.6 tps, lat 1685.074 ms stddev 184.830
progress: 100.0 s, 180.0 tps, lat 1577.312 ms stddev 57.277
progress: 110.0 s, 164.4 tps, lat 1785.914 ms stddev 198.351
progress: 120.0 s, 171.8 tps, lat 1685.099 ms stddev 151.035

transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: extended
number of clients: 290
number of threads: 2
duration: 3600 s
number of transactions actually processed: 567914
latency average = 1832.948 ms
latency stddev = 245.638 ms
average connection time = 5.359 ms
tps = 157.714403 (including reconnection times)


==============================================================================================================================================================
-c клиенты
--client=клиенты
Число имитируемых клиентов, то есть число одновременных сеансов базы данных. 

-C
--connect
Устанавливать новое подключение для каждой транзакции вместо одного для каждого клиента. Это полезно для оценивания издержек подключений.

-j потоки
--jobs=потоки
Число рабочих потоков в pgbench. Использовать нескольких потоков может быть полезно на многопроцессорных компьютерах. Клиенты распределяются по доступным потокам равномерно, насколько это возможно.

-P сек
--progress=сек
Выводить отчёт о прогрессе через заданное число секунд (сек). Выдаваемый отчёт включает время, 
прошедшее с момента запуска, скорость (в TPS) с момента предыдущего отчёта, а также среднее время ожидания транзакций и стандартное отклонение. 
В режиме ограничения скорости (-R) время ожидания вычисляется относительно назначенного времени запуска 
транзакции, а не фактического времени её начала, так что оно включает и среднее время отставания от графика.


-T секунды
--time=секунды
Выполнять тест с ограничением по времени (в секундах), а не по числу транзакций для каждого клиента. Параметры -t и -T являются взаимоисключающими.

-M режим_запросов
--protocol=режим_запросов
Протокол, выбираемый для передачи запросов на сервер:
simple: использовать протокол простых запросов.
extended: использовать протокол расширенных запросов.
prepared: использовать протокол расширенных запросов с подготовленными операторами.
==============================================================================================================================================================



теперь в логе первой ноды видим что чтобы докатиться до мастера ноде необходимы вал сегменты которые уже были удалены(презатерты)


мы останавливаем патрони 
sudo systemctl stop patroni.service
sudo systemctl status patroni.service

удаляем все из директории с данными postgres 
rm -rf /var/lib/pgsql/14/data/*

и запускаем патрони 
sudo systemctl start patroni.service
sudo systemctl status patroni.service


если все сделано верно
будет выполненно пресоздание первой ноды
</pre>





<pre>
часть 7 backup
==========================================================================================================================================================================
==========================================================================================================================================================================
==========================================================================================================================================================================
==========================================================================================================================================================================
==========================================================================================================================================================================
мой преработанный скрипт получить ip адрес ноды лидера

#!/bin/bash
# Скрипт выполнения бекапа для баз Postgres уттилита pg_basebackup (обязательно поменять порт)
#

#    Доступны следующие опции:
#    -a | --all      - создание резервной FULL копии и выполнение rebild index and vaccum на всей базе данных выполняется в воскресенье
#    -d | --day      - создание резервной FULL копии без rebild index and vaccum выполняется ежедневно
#    Пред началом backup Проверяется какой узел в кластере является Лидером если ip адрес лидера не совпадает с ip адресом сервера на котором запускается скрипт то backup не осуществляется.

usage=" ==================================================================================================\n
        backup Postgres v 1.0\n
        use \n
        -d or --day         -  backup every day but Sunday ( not rebild index and vaccum )\n
        -a or --all         -  backup and rebild index and vaccum on Sunday\n
        =================================================================================================\n
        "


day_backup=0
sunday_day_backup=0

if [[ -z "$1" ]] ;  then
    echo  -e $usage
    exit 1
fi

while [ "$1" != "" ]; do
    case $1 in
        -d | --day )            day_backup=777
                                ;;
        -a | --all )            sunday_day_backup=888
                                ;;
        -h | --help )      echo -e $usage
                           exit
                           ;;
        *)                 echo "unknown parameter"
                           echo  -e $usage
                           exit 1 
                           ;;
    esac
    shift
done


ipaddr=`ifconfig | sed -En 's/127.0.0.1//;s/.*inet (addr:)?(([0-9]*\.){3}[0-9]*).*/\2/p'`
node_Leader=$(patronictl -c /etc/patroni/patroni.yml list | grep " pg_" | grep -v "Replica" | head -1 | awk -F"|" '{print $3}' | awk '{print $1}')
namehost=`hostname`
select_version='SELECT version();'
pg_sql=`psql -U postgres -d postgres -c "$select_version"`
server_version=`echo $pg_sql | sed 's/-//g' | awk '{print substr($0, 1, length($0) -8)}'`
version=`echo $server_version | head -c 23`
postgre_rebild_index=`find /usr/pgsql* -iname 'reindexdb'`
postgre_vacuumdb=`find /usr/pgsql* -iname 'vacuumdb'`
#директория с логами выполнения backup
slog=/var/log/postgres_backup_log/postgres_backup.log
#пользователь который выполняет бекап
dbUser=postgres
# путь к директории резервной копии
DESTDIR=/var/postgre_backup


echo "ipaddr $ipaddr"
echo "node_Leader $node_Leader"
#Проверяется какой узел в кластере является Лидером
if [ "$ipaddr" == "$node_Leader" ];
then
        echo ""$ipaddr" == "$node_Leader""
        if [[ $day_backup -eq "777" ]]
  	then
        	# Записываем информацию в лог с секундами
    		echo "Запуск резервного копирования кластера ежедневно кроме воскресенья"
    		echo "-----------------------------------------------------------------------------------------" >> ${slog}
    		echo "`date +"%Y-%m-%d_%H-%M-%S"` Запуск резервного копирования кластера $namehost $ipaddr" >> ${slog}
    		echo "$server_version" >> ${slog}

    		su - postgres -c "/usr/bin/pg_basebackup -U postgres -p 5432 --progress --verbose --format=tar --gzip --pgdata=$DESTDIR/all_db_postgre_backup" >> ${slog}

    		RETURN_STATUS=$?
    		if [ $RETURN_STATUS -eq 0 ]
    		then
        		echo "`date +"%Y-%m-%d_%H-%M-%S"` Резервное копирование кластера баз $namehost $ipaddr" выполнено >> ${slog}

        		# отправка почты админам
        		echo sending a letter to administrators about the successful backup database posrgres
        		mailbody="$version. PROCESS EXIT WITH CODE $RETURN_STATUS $namehost $ipaddr"
        		echo "From: xxxx@xxxxx.ttttt.srv" > /tmp/mailtest
        		echo "To: my_mail@yandex.xxxx.srv" >> /tmp/mailtest
        		echo "Subject:$namehost $ipaddr $version BACKUP HAS BEEN CREATED WITH SUCCESS" >> /tmp/mailtest
        		echo $mailbody3 >> /tmp/mailtest
        		echo $mailbody >> /tmp/mailtest
        		cat /tmp/mailtest | /usr/sbin/sendmail -t
        		exit
    	else
        	echo "`date +"%Y-%m-%d_%H-%M-%S"` FAIL backup кластера $version $namehost $ipaddr error"  >> ${slog}
        	
        	# отправка почты админам
        	echo "FAIL backup database posrgres. ALARM !!!!!!!!!!!!!!!!!!"
        	mailbody="$version FAIL backup.  process failed. Exit with code $RETURN_STATUS $namehost $ipaddr"
        	echo "From: xxxx@xxxxx.ttttt.srv" > /tmp/mailtest
        	echo "To: my_mail@yandex.xxxx.srv" >> /tmp/mailtest
        	echo "Subject:$namehost $ipaddr $version BACKUP FAILED" >> /tmp/mailtest
        	echo $mailbody >> /tmp/mailtest
        	cat /tmp/mailtest | /usr/sbin/sendmail -t
        	echo "An error occured. Database backup failed. Exit with code $RETURN_STATUS"
        	exit
    		fi
	fi





	#выполняется в воскресенье
	if [[ $sunday_day_backup -eq "888" ]]
	then
    		# Записываем информацию в лог с секундами
    		echo "Запуск резервного копирования кластера в воскресенье"
    		echo "-----------------------------------------------------------------------------------------" >> ${slog}
    		echo "`date +"%Y-%m-%d_%H-%M-%S"` Запуск резервного копирования кластера $namehost $ipaddr" >> ${slog}
    		echo "$server_version" >> ${slog}
		
    		su - postgres -c "/usr/bin/pg_basebackup -U postgres -p 5432 --progress --verbose --format=tar --gzip --pgdata=$DESTDIR/all_db_postgre_backup" >> ${slog}
		
    		RETURN_STATUS=$?
    		if [ $RETURN_STATUS -eq 0 ]
    		then
        		echo "`date +"%Y-%m-%d_%H-%M-%S"` Резервное копирование кластера баз $namehost $ipaddr" выполнено >> ${slog}
        		echo "$server_version" >> ${slog}
			
        		# отправка почты админам
        		echo sending a letter to administrators about the successful backup database posrgres
        		mailbody="$version. PROCESS EXIT WITH CODE $RETURN_STATUS $namehost $ipaddr"
        		echo "From: xxxx@xxxxx.ttttt.srv" > /tmp/mailtest
        		echo "To: my_mail@yandex.xxxx.srv" >> /tmp/mailtest
        		echo "Subject:$namehost $ipaddr $version BACKUP HAS BEEN CREATED WITH SUCCESS" >> /tmp/mailtest
        		echo $mailbody3 >> /tmp/mailtest
        		echo $mailbody >> /tmp/mailtest
        		cat /tmp/mailtest | /usr/sbin/sendmail -t
			
        		# Переиндексировать базу
        		echo "Запуск реиндексации всех баз --all"
        		echo "`date +"%Y-%m-%d_%H-%M-%S"` Запуск реиндексации баз кластера $namehost $ipaddr" >> ${slog}
        		$postgre_rebild_index -U $dbUser -p 5432 --all >> ${slog}
        		echo "`date +"%Y-%m-%d_%H-%M-%S"` Реиндексация всех баз кластера выполнена" >> ${slog}
        		echo "*****************************************************************************************" >> ${slog}
			
        		#Выполняем очистку и анализ базы данных
        		echo "Запуск очистки всех баз --all"
        		echo "`date +"%Y-%m-%d_%H-%M-%S"` Запуск очистки баз $namehost $ipaddr" >> ${slog}
        		$postgre_vacuumdb -U $dbUser -p 5432 --full --analyze --all >> ${slog}
        		echo "`date +"%Y-%m-%d_%H-%M-%S"` Очистка всех баз кластера выполнена" >> ${slog}
        		echo "-----------------------------------------------------------------------------------------" >> ${slog}
        		echo "                                                                                         " >> ${slog}
        		echo "-" >> ${slog}
        		exit
    		else    
        		echo "`date +"%Y-%m-%d_%H-%M-%S"` FAIL backup кластера $version $namehost $ipaddr error"  >> ${slog}
        		
        		echo FAIL backup database posrgres. ALARM !!!!!!!!!!!!!!!!!!
        		mailbody="FAIL backup $version.  process failed. Exit with code $RETURN_STATUS $namehost $ipaddr"
        		echo "From: xxxx@xxxxx.ttttt.srv" > /tmp/mailtest
        		echo "To: my_mail@yandex.xxxx.srv" >> /tmp/mailtest
        		echo "Subject:BACKUP FAILED $namehost $ipaddr $version BACKUP FAILED" >> /tmp/mailtest
        		echo $mailbody >> /tmp/mailtest
        		cat /tmp/mailtest | /usr/sbin/sendmail -t
        		echo "An error occured. Database backup failed. Exit with code $RETURN_STATUS"
        		exit
    		fi
	else
    		echo "`date +"%Y-%m-%d_%H-%M-%S"` FAIL backup кластера $version $namehost $ipaddr error"  >> ${slog}
    		echo "FAIL backup database posrgres. ALARM7777777777 !!!!!!!!!!!!!!!!!!"
    		mailbody="Postgres $version error occured.  process failed. Exit with code $RETURN_STATUS $namehost $ipaddr"
    		echo "From: xxxx@xxxxx.ttttt.srv" > /tmp/mailtest
    		echo "To: my_mail@yandex.xxxx.srv" >> /tmp/mailtest
    		echo "Subject:BACKUP FAILED $namehost $ipaddr $version" >> /tmp/mailtest
    		echo $mailbody >> /tmp/mailtest
    		cat /tmp/mailtest | /usr/sbin/sendmail -t
    		echo "An error occured. Database backup failed. Exit with code $RETURN_STATUS"
    		exit
	fi			
else
        echo  ""$ipaddr" != "$node_Leader"" 
fi


</pre>



<pre>
ссылка на презентацию
=============================================================================================================================================================================

https://docs.google.com/presentation/d/1ugjXX1yX400tUXxBIZaLgiwlJwpl3EzQ5fF2sp3wXPY/edit#slide=id.g1b638906324_0_66
https://docs.google.com/presentation/d/1ugjXX1yX400tUXxBIZaLgiwlJwpl3EzQ5fF2sp3wXPY/edit#slide=id.g1b638906324_0_66
https://docs.google.com/presentation/d/1ugjXX1yX400tUXxBIZaLgiwlJwpl3EzQ5fF2sp3wXPY/edit#slide=id.g1b638906324_0_66

</pre>






<pre>
Черновик
==================================================================================================================================================



В конфигурации haproxy есть два важных раздела прослушивания :
первичный , использующий порт 5000 для запросов на чтение/запись.
в режиме ожидания с использованием порта 5001 только для запросов на чтение.
Все три узла включены в оба раздела: это потому, что все серверы баз 
данных являются потенциальными кандидатами на роль первичных или вторичных. 
Patroni предоставляет встроенную поддержку REST API для мониторинга проверки работоспособности, 
которая отлично работает с HAProxy. HAProxy отправит HTTP-запрос на порт 8008 патрони, чтобы узнать, какую роль в настоящее время имеет каждый узел.



HAProxy в этом случае будет выступать не только как прокси, 
но и как балансировщик нагрузки. Мы можем разделить соединение для чтения и записи на отдельные порты.

Вам нужен первый вариант, потому что pgBouncer — это пул соединений для PostgreSQL. 
Он должен быть близок к СУБД и быть один на один подключен каждый к своему экземпляру Postgresql. 
Что касается HAProxy, это балансировщик нагрузки, у вас может быть номер HAProxy, 
отличный от количества экземпляров Postgre\Patroni\pgBouncer. Кроме того, 
это даст вам возможность разделять запросы на чтение и запись в разные порты\базы данных.



Была реализованна схема полключения клиентов к PostgreSQL кластеру:
Клиенты (Приложение) > HAproxy > Pgbouncer > PostgreSQL (Patroni)

идем по принцепу:
Приложение > HAproxy > Pgbouncer > PostgreSQL (Patroni)
	
</pre>	

	
	
	
	
	


