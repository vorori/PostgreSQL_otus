крутые статьи
https://goteleport.com/blog/running-postgresql-on-kubernetes/
https://goteleport.com/blog/running-postgresql-on-kubernetes/
https://goteleport.com/blog/running-postgresql-on-kubernetes/

https://www.citusdata.com/product/community
https://www.citusdata.com/product/community
https://www.citusdata.com/product/community


черновик
----------------------------------------------------------
kubectl get all
kubectl get all -A
kubectl get pods
kubectl get nodes
kubectl get pv
kubectl get all -o wide

-- посмотрим дефолтный тип стораджа
kubectl get storageclasses

kubectl get pvc -o wide
kubectl get pv -o wide
----------------------------------------------------------


сделать алиасы в файле .baashrc для упрошения ввода команд
----------------------------------------------------------
# some more ls aliases
alias ll='ls -alf'
alias la='ls -A'
alias l='ls -CF'
alias gc='cd /mnt/d/download/5homework; sudo git clone'
alias k='kubectl'
alias kg='kubectl get all'
alias kga='kubectl get all -A' alias ka 'kubectl apply -f
alias kd='kubectl delete -f .'
alias kc='kubectl describe'
alias h='helm'
alias nr='newman run'
----------------------------------------------------------



----------------------------------------------------------------------------------------------------------------------------------------------------
----------------------------------------------------------примеры конфигураций для создания диска---------------------------------------------------
----------------------------------------------------------------------------------------------------------------------------------------------------

----------------------------------------------------------
про то как подключать диск а именно pv PersistentVolume 
расказывает на 40 минуте -44 минута
-----------------
1)пример создания заявки на PersistentVolume:

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
	name: pvc-demo
spec:
  accessModes:
	- ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
-----------------
-----------------
2)а тут мы уже нашим преложением подвязывемся к сушествуюшему pv

kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: postgres-pv-claim  # в моем случае pvc-demo!!!!!!
  labels:
    app: postgres
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 5Gi
-----------------
----------------------------------------------------------


----------------------------------------------------------
пример запуска postgres со всеми настройками кторые нам нужны видео 49 минута
kubectl apply -f postgres-configmap.yaml -f postgres-storage.yaml -f postgres-deployment.yaml -f postgres-service.yaml
kubectl apply -f postgres-configmap.yaml -f postgres-storage.yaml -f postgres-deployment.yaml -f postgres-service.yaml
kubectl apply -f postgres-configmap.yaml -f postgres-storage.yaml -f postgres-deployment.yaml -f postgres-service.yaml


----------------------------------------------------------------------------------------------------------------------------------------------------
----------------------------------------------------------примеры конфигураций:---------------------------------------------------------------------
----------------------------------------------------------------------------------------------------------------------------------------------------

58:46
----------------------------------------------------------
файлик postgres-deployment.yaml
-----------------
apiVersion: apps/v1 
kind: Deployment 
metadata:
   name: postgres
spec:
  replicas: 1 
  selector:
     matchLabels:
       app: postgres
  template:
    metadata:
     Labels:
	  app: postgres
    spec:
     containers:
	   - name: postgres image: postgres:14
         imagePullPolicy: "IfNotPresent"
         ports:
- containerPort: 5432
envFrom:
- configMapRef:
name: postgres-config
volumeMounts:
- mountPath: /var/lib/postgresql
name: postgredb
-----------------

----------------------------------------------------------


----------------------------------------------------------
файлик postgres-service.yaml
-----------------
apiVersion: v1 
kind: Service 
metadata:
   name: postgres 
   Labels:
     app: postgres
spec:
  type: NodePort 
  ports:
  - port: 5432 
  selector:
    app: postgres
-----------------

----------------------------------------------------------



преновначальная настройка ОС

1)
#Выполняю наименование своих четырех машинок 
Изменить имя хоста с помощью hostnamectl
Чтобы установить постоянное имя хоста с помощью команды hostnamectl, используйте команду.
sudo hostnamectl set-hostname kubmaster --static
sudo hostnamectl set-hostname kub1 --static
sudo hostnamectl set-hostname kub2 --static
sudo hostnamectl set-hostname kub3 --static

1.1)
#в моем случае днс сервера нет поэтому я делаю это вручную
sudo hostnamectl set-hostname kubmaster --transient
sudo hostnamectl set-hostname kub1 --transient
sudo hostnamectl set-hostname kub2 --transient
sudo hostnamectl set-hostname kub3 --transient

2)
Подтвердите свое новое имя хоста.
hostnamectl 

3)
Этот параметр автоматически обновит файл /etc/hostname.
cat /etc/hostname 


часть 1 Hyper-V окружение
==========================================================================================================================================================================
==========================================================================================================================================================================
==========================================================================================================================================================================
==========================================================================================================================================================================
==========================================================================================================================================================================
1)
создал сеть остнастка диспетчер виртуальных комутаторов "VM and PC" для обединения рабочей станции с серверами 
сеть 192.168.2.0 маска 255.255.255.0


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
hostname kubmaster ip  192.168.2.10
hostname kub1      ip  192.168.2.11
hostname kub2      ip  192.168.2.12
hostname kub3      ip  192.168.2.13


5)
Запускаем каждый сервер и делаем настройки
настройка сеть на каждом сервере linux 

пример:
cat <<EOF | > sudo tee /etc/sysconfig/network-scripts/ifcfg-eth0
TYPE=Ethernet
HWADDR=00:15:5d:02:06:2e
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=none
IPADDR=192.168.2.13
PREFIX=24
GATEWAY=192.168.2.1
DNS1=192.168.2.1
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=no
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=eth0
UUID=2aa5d1b2-e909-4e4c-b697-43de691d7bf2
DEVICE=eth0
ONBOOT=ye
EOF



vim /etc/sysconfig/network-scripts/ifcfg-eth0
systemctl restart network




==========================================================================================================================================================================


hostname && hostname -i
hostname && hostname -i
hostname && hostname -i


1) добавляю на каждой ноде 
sudo cat <<EOF>> /etc/hosts
192.168.2.10 kubmaster
192.168.2.11 node1 kub1
192.168.2.12 node2 kub2
192.168.2.13 node3 kub3
EOF

hostname kubmaster ip  192.168.2.10
hostname kub1      ip  192.168.2.11
hostname kub2      ip  192.168.2.12
hostname kub3      ip  192.168.2.13


Шаг 1: Предварительные требования 
1.a.. Проверьте ОС, конфигурацию оборудования и Сетевое подключение 
1.b.. Отключите подкачку Disable SWAP и брандмауэр 

sudo yum -y install epel-release
yum install -y htop mc vim wget telnet

sudo sed -i '/swap/d' /etc/fstab && sudo swapoff -a 
sudo systemctl stop firewalld 
sudo systemctl disable firewalld 

1.c Отключить Selinux 
Контейнеры должны получить доступ к файловой системе хоста. SELinux должен быть
установлен в разрешающий режим, который эффективно отключает его функции безопасности.


sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
sudo setenforce 0


Шаг 2. Настройте локальные таблицы IP для просмотра мостового трафика 
2.a.. Включите мостовой трафик 

sudo modprobe br_netfilter && lsmod | grep br_netfilter

2.b.. Скопируйте приведенное ниже содержимое в этот файл.. /etc/modules-load.d/k8s.conf
cat <<EOF | > sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF


Скопируйте приведенное ниже содержимое в этот файл.. /etc/sysctl.d/k8s.conf 
cat <<EOF | > sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF


sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF


# Ensure you load modules
sudo modprobe overlay && sudo modprobe br_netfilter && sudo sysctl --system



Шаг 3. Установите Docker будем работать через его движок

3.a.. Удалите все старые версии если чтото было установлено
sudo yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine

3. б.. Установите утилиты Yum | Диспетчер конфигурации  
sudo yum install -y yum-utils

3.c.. Настройте репозиторий Docker
yum-config-manager  --add-repo https://download.docker.com/linux/centos/docker-ce.repo

cat <<EOF | > sudo tee /etc/yum.repos.d/docker.repo
[docker]
baseurl=https://download.docker.com/linux/centos/7/x86_64/stable/
gpgcheck=0
EOF

3. d.. Установите Docker Engine, Docker CLI, Docker RUNTIME $ 
yum install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
yum install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin


4.b. Скопируйте приведенное ниже содержимое в этот файл.. /etc/docker/
sudo tee /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF


sudo systemctl enable docker && sudo systemctl restart docker && sudo systemctl status docker
sudo systemctl daemon-reload


Шаг 5. Установите kubeadm, kubectl, kubelet 
5.a.. Скопируйте в этот файл приведенное ниже содержимое.. /etc/yum.repos.d/kubernetes.repo
cat <<EOF | > sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

смотрим какие версии доступны для установки
yum --showduplicates list kubeadm.x86_64
yum --showduplicates list kubeadm.x86_64
yum --showduplicates list kubeadm.x86_64


Устанавливаем командой new
yum install -y kubelet-1.21.0-0.x86_64 kubeadm-1.21.0-0.x86_64 kubectl-1.21.0-0.x86_64
yum install -y kubelet-1.21.0-0.x86_64 kubeadm-1.21.0-0.x86_64 kubectl-1.21.0-0.x86_64
yum install -y kubelet-1.21.0-0.x86_64 kubeadm-1.21.0-0.x86_64 kubectl-1.21.0-0.x86_64

Устанавливаем командой old
yum install -y kubelet-1.16.2-0.x86_64 kubeadm-1.16.2-0.x86_64 kubectl-1.16.2-0.x86_64


Я использовал следующую команду для создания конфигурации по умолчанию (при новой установке на master):
kubeadm init --pod-network-cidr=10.10.0.0/16 --apiserver-advertise-address=master_nodeIP
kubeadm init --pod-network-cidr="10.10.0.0/16" --apiserver-advertise-address="10.128.0.20"
kubeadm init --pod-network-cidr="10.10.0.0/16" --apiserver-advertise-address="10.128.0.20"
или
kubeadm init phase kubelet-start
kubeadm init phase kubelet-start

где:
--pod-network-cidr=10.10.0.0/16 - диапазон сети pod
--apiserver-advertise-address=master_nodeIP  - ip адрес кластера(главного узла кластера) вставляем сюда ip VM на которой инициализируем кластер

кластер готов
----------------------------------------------------------------------------------------------------------------------------
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.128.0.20:6443 --token mlmf5i.6la6cbrxy9via93q \
        --discovery-token-ca-cert-hash sha256:e596f8d4c4d599c68b1bb0beee8bc2885aa87c792f3f83e1d0e91dedbeac38fc
		
----------------------------------------------------------------------------------------------------------------------------


добавляем в автозагрузку
sudo systemctl enable --now kubelet && systemctl status kubelet
sudo systemctl enable --now kubelet && systemctl status kubelet
sudo systemctl enable --now kubelet && systemctl status kubelet

версия
kubectl version
kubectl version
kubectl version


Управляйте кластером как обычный пользователь
Чтобы начать использовать кластер, вам нужно запустить его как обычный пользователь, набрав:
Чтобы начать использовать свой кластер, вам необходимо запустить следующее как обычный пользователь: 
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config



Настройте сеть Pod
Pod Network позволяет узлам внутри кластера взаимодействовать. 
Существует несколько доступных сетевых вариантов Kubernetes. Используйте следующую команду для установки сетевой надстройки flannel pod:
https://kubernetes.io/docs/concepts/cluster-administration/networking/#how-to-implement-the-kubernetes-networking-model

https://www.weave.works/docs/net/latest/kubernetes/kube-addon/#install
https://www.weave.works/docs/net/latest/kubernetes/kube-addon/#install


# Запуск только в главном узле 
# Запускать только в мастер-ноде настроим сеть для подов
для Weave Networks:
с помошью этой команды будет установлен плагин Weave со всеми необходимыми разрешениями для кластера
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml


Шаг 9. Присоедините рабочие узлы к мастеру
Присоедините рабочий узел к кластеру
Как указано в шаге 1 , вы можете использовать kubeadm join команду на каждом рабочем узле, чтобы подключить его к кластеру.
Замените коды на коды с вашего главного сервера. Повторите это действие для каждого рабочего узла в вашем кластере.
# Запуск в рабочих узлах как "Root" 

kubeadm join 10.128.0.20:6443 --token mlmf5i.6la6cbrxy9via93q \
        --discovery-token-ca-cert-hash sha256:e596f8d4c4d599c68b1bb0beee8bc2885aa87c792f3f83e1d0e91dedbeac38fc

где:
10.128.0.20:6443 -- адрес главного узла куба мастер ноды !!!!

после запуска команды на присоединение мы мониторим когда рабочий узел кластера будет готов
после запуска команды на присоединение мы мониторим когда рабочий узел кластера будет готов
после запуска команды на присоединение мы мониторим когда рабочий узел кластера будет готов
чтобы получить информацию о количестве узлов в кластере(мы должны увидеть что в кластере доступен только главный узел)!!!!!
kubectl get nodes
kubectl get nodes
kubectl get nodes


vorori@masterkubernetes ~]$ kubectl get nodes
NAME                                    STATUS   ROLES                  AGE     VERSION
kb1.ru-central1.internal                Ready    <none>                 61s     v1.21.0
kb2.ru-central1.internal                Ready    <none>                 56s     v1.21.0
kb3.ru-central1.internal                Ready    <none>                 49s     v1.21.0
masterkubernetes.ru-central1.internal   Ready    control-plane,master   7m55s   v1.21.0


выполнил на kвсех присоединенных нодах
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config     ### копирую клюс с мастера vim $HOME/.kube/config    ### cat $HOME/.kube/config 
sudo chown $(id -u):$(id -g) $HOME/.kube/config


проверяю на каждой ноде
[vorori@kb3 ~]$ kubectl get nodes
NAME                                    STATUS   ROLES                  AGE     VERSION
kb1.ru-central1.internal                Ready    <none>                 7m52s   v1.21.0
kb2.ru-central1.internal                Ready    <none>                 7m47s   v1.21.0
kb3.ru-central1.internal                Ready    <none>                 7m40s   v1.21.0
masterkubernetes.ru-central1.internal   Ready    control-plane,master   14m     v1.21.0



добавляю на кадлой ноде:
sudo cat <<EOF>> /etc/hosts
10.128.0.20 masterkubernetes.ru-central1.internal
10.128.0.10 node1 kb1.ru-central1.internal
10.129.0.28 node2 kb2.ru-central1.internal
10.129.0.11 node3 kb3.ru-central1.internal
EOF


проверяю доступность кластера 
kubectl cluster-info
kubectl cluster-info
kubectl cluster-info


kubectl get pods --namespace kube-system
get pods --namespace kube-system
get pods --namespace kube-system


kubectl get pods
kubectl get pods
kubectl get pods

описание:
Когда приложение работает правильно, каждый из модулей должен иметь:
Значение 1/1 в колонке ГОТОВО
Значение Running в столбце СТАТУС

В выходных данных приведенного выше примера модули сделать выводв названии создаются при развертывании модели. 
Они появятся только в том случае, если в экземпляре приложения, работающего в системе, развернуты модели.
Значение STATUS, отличное от Running указывает на проблему с модулем.
Ненулевое и увеличивающееся значение в столбце RESTARTS указывает на проблему с этим модулем.


Чтобы установить роль для вашего рабочего узла, используйте следующую команду:
sudo kubectl label node kb1.ru-central1.internal node-role.kubernetes.io/worker=worker







------------------------------------------------------------------------------------------------------------------------------------------------------------------
------------------------------------------------------------------------------------------------------------------------------------------------------------------
------------------------------------------------------------------------------------------------------------------------------------------------------------------
------------------------------------------------------------------------------------------------------------------------------------------------------------------
------------------------------------------------------------------------------------------------------------------------------------------------------------------
https://www.cockroachlabs.com/docs/stable/deploy-cockroachdb-with-kubernetes.html?filters=helm
https://www.cockroachlabs.com/docs/stable/deploy-cockroachdb-with-kubernetes.html?filters=helm

https://habr.com/ru/companies/flant/articles/328756/
https://habr.com/ru/companies/flant/articles/328756/


Развернуть CockroachDB в GKE или GCE
Потесировать dataset с чикагскими такси
Или залить 10Гб данных и протестировать скорость запросов в сравнении с 1 инстансом PostgreSQL
Описать что и как делали и с какими проблемами столкнулись


скачиваю
wget https://raw.githubusercontent.com/cockroachdb/cockroach/master/cloud/kubernetes/cockroachdb-statefulset.yaml

редактирую конфиг под себя
vim cockroachdb-statefulset.yaml
storage: 15Gi


запускаю
kubectl create -f cockroachdb-statefulset.yaml
kubectl create -f cockroachdb-statefulset.yaml

вывод после запуска
service/cockroachdb-public created
service/cockroachdb created
Warning: policy/v1beta1 PodDisruptionBudget is deprecated in v1.21+, unavailable in v1.25+; use policy/v1 PodDisruptionBudget
poddisruptionbudget.policy/cockroachdb-budget created
statefulset.apps/cockroachdb created



удалить 
kubectl get all
kubectl get pod
kubectl delete statefulset.apps/cockroachdb

Helm install
Helm дает вам быстрый и простой способ развернуть экземпляр PostgreSQL в вашем кластере.
скачиваем распаковываем
wget https://get.helm.sh/helm-v3.7.0-linux-amd64.tar.gz && \
tar xvf helm-v3.7.0-linux-amd64.tar.gz

Переместите двоичные файлы в локальный каталог двоичных файлов
sudo mv linux-amd64/helm /usr/local/bin

Очистка
rm helm-v3.7.0-linux-amd64.tar.gz && \
rm -rf linux-amd64

3.5)
проверяем версию
[root@localhost tmp]# helm version
version.BuildInfo{Version:"v3.7.0", GitCommit:"eeac83883cb4014fe60267ec6373570374ce770b", GitTreeState:"clean", GoVersion:"go1.16.8"}


[vorori@masterkubernetes ~]$ helm repo add cockroachdb https://charts.cockroachdb.com/
"cockroachdb" has been added to your repositories


[vorori@masterkubernetes ~]$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "cockroachdb" chart repository
Update Complete. ⎈Happy Helming!⎈


устанавливаем cockroachdb
helm install my-release --values /home/vorori/my-values.yaml cockroachdb/cockroachdb


https://github.com/cockroachdb/helm-charts/blob/master/cockroachdb/values.yaml
https://github.com/cockroachdb/helm-charts/blob/master/cockroachdb/values.yaml
https://github.com/cockroachdb/helm-charts/blob/master/cockroachdb/values.yaml


https://github.com/cockroachdb/helm-charts/tree/master/cockroachdb
https://github.com/cockroachdb/helm-charts/tree/master/cockroachdb
https://github.com/cockroachdb/helm-charts/tree/master/cockroachdb


Создайте локальный файл YAML (например, my-values.yaml), чтобы указать свои пользовательские значения. 
Они будут использоваться для переопределения значений по умолчанию в values.yaml.


helm install my-release666 cockroachdb/cockroachdb
helm install my-release666 cockroachdb/cockroachdb
helm install my-release666 cockroachdb/cockroachdb

перечислить все версии cockroachdb/
helm search repo -l cockroachdb/cockroachdb
helm search repo -l cockroachdb/cockroachdb
helm search repo -l cockroachdb/cockroachdb






cockroachdb/cockroach:v23.1.4


наблюдаем за установкой
[vorori@kb1 ~]$ kubectl get pod
NAME                     READY   STATUS    RESTARTS   AGE
cockroachdb-0            0/1     Pending   0          2m50s
cockroachdb-1            0/1     Pending   0          2m50s
cockroachdb-2            0/1     Pending   0          2m50s
cockroachdb-init-95852   1/1     Running   0          2m50s



показать логи podname
[vorori@kb1 ~]$ kubectl logs cockroachdb-0
[vorori@kb1 ~]$ kubectl logs cockroachdb-init-95852