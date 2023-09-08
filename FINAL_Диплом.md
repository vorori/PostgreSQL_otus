<pre>
---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
----------------------------------мануал кластера k8s в яндекс облаке------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
</pre>



#### подготовительные мероприятия нарезал vm установил необходимые пакеты

#### получить данные vm

<pre>
hostname && hostname -i
hostname && hostname -i
hostname && hostname -i
</pre>


#### 1) 

#### подготовка наименование машин
#### добавляю на каждой ноде 

<pre>
sudo cat <<EOF>> /etc/hosts
10.129.0.24 masterkub.ru-central1.internal
10.128.0.6 node1 kub1.ru-central1.internal
10.129.0.19 node2 kub2.ru-central1.internal
10.130.0.34 node3 kub3.ru-central1.internal
EOF

#### самопроверка
cat /etc/hosts
cat /etc/hosts


#### устанавливаю пакет которые нужны в работе
sudo yum -y install epel-release && yum install -y htop mc vim wget telnet git
sudo yum -y install epel-release && yum install -y htop mc vim wget telnet git
sudo yum -y install epel-release && yum install -y htop mc vim wget telnet git

</pre>


#### 1.1)
#### Предварительные требования 
#### 1.a.. Проверяю ОС, конфигурацию оборудования и Сетевое подключение  ( смотрим ошибки в логах ОС на раннем этапе)
#### 1.b.. Отключаю подкачку Disable SWAP настраиваем брандмауэр ( правила описанны ниже)
#### 1.с.. препроверяю файл подкачки  С помощью команды swapon: (Если команда ничего не возвращает, значит файла подкачки не существует)


<pre>
swapon -s
sudo sed -i '/swap/d' /etc/fstab && sudo swapoff -a 
cat /etc/fstab
sudo systemctl stop firewalld 
sudo systemctl disable firewalld 
</pre>

#### 1.d Отключить Selinux 
#### Контейнеры должны получить доступ к файловой системе хоста. SELinux должен быть
#### установлен в разрешающий режим, который эффективно отключает его функции безопасности.

<pre>
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config && sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config && sudo setenforce 0


#### самопроверка
cat /etc/selinux/config
cat /etc/selinux/config
</pre>

#### 2)

#### Настроим локальные таблицы IP для просмотра мостового трафика 
#### Включаем мостовой трафик 

<pre>
sudo modprobe br_netfilter && lsmod | grep br_netfilter
sudo modprobe br_netfilter && lsmod | grep br_netfilter
</pre>

#### k8s
#### 2.a.. копирую приведенное ниже содержимое в этот файл.. /etc/modules-load.d/k8s.conf

<pre>
cat <<EOF | > sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF
</pre>

#### проверка
<pre>
cat /etc/modules-load.d/k8s.conf
cat /etc/modules-load.d/k8s.conf
</pre>

#### k8s
#### копирую приведенное ниже содержимое в этот файл .. /etc/sysctl.d/k8s.conf 

<pre>
cat <<EOF | > sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
</pre>

#### проверка

<pre>
cat /etc/sysctl.d/k8s.conf 
cat /etc/sysctl.d/k8s.conf 
</pre>

#### копирую приведенное ниже содержимое в этот файл .. /etc/sysctl.d/k8s.conf 
<pre>
cat <<EOF | > sudo tee /etc/sysctl.d/kubernetes.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
</pre>

#### проверка

<pre>
cat /etc/sysctl.d/kubernetes.conf
cat /etc/sysctl.d/kubernetes.conf
</pre>


#### проверяю что модули загружены
<pre>
sudo modprobe overlay
sudo modprobe br_netfilter
sysctl --system
</pre>

<pre>
sudo modprobe overlay && sudo modprobe br_netfilter && sudo sysctl --system
sudo modprobe overlay && sudo modprobe br_netfilter && sudo sysctl --system
sudo modprobe overlay && sudo modprobe br_netfilter && sudo sysctl --system
</pre>

<pre>
reboot -h now
reboot -h now
reboot -h now

tail -f /var/log/messages
tail -f /var/log/messages
tail -f /var/log/messages

Настройка контейнера

cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

cat /etc/modules-load.d/containerd.conf
cat /etc/modules-load.d/containerd.conf
cat /etc/modules-load.d/containerd.conf

sudo modprobe overlay && sudo modprobe br_netfilter
sudo modprobe overlay && sudo modprobe br_netfilter
sudo modprobe overlay && sudo modprobe br_netfilter

# Setup required sysctl params, these persist across reboots.
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system

</pre>

#### 3)
#### установлю Docker будем работать через его движок
#### удаляю все старые версии если чтото было установлено

<pre>
sudo yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine
</pre>

#### 3. a.. устанавливаю утилиты Yum

<pre>
sudo yum install -y yum-utils
</pre>

####  3.b.. настраиваю репозиторий Docker

<pre>
yum-config-manager  --add-repo https://download.docker.com/linux/centos/docker-ce.repo
</pre>

#### 3.c.. устанавливаю Docker Engine, Docker CLI, Docker RUNTIME 

<pre>
yum install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

#Start containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml


sudo systemctl restart containerd && sudo systemctl enable containerd && systemctl status containerd
</pre>

#### 3.d. для работы docker копирую е приведенное ниже содержимое в этот файл.. /etc/docker/  

<pre>
cat <<EOF | > sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
</pre>

#### проверка
<pre>
cat /etc/docker/daemon.json
</pre>

#### 4)

#### стартуем docker с добавляем в автозагрузку

<pre>
sudo systemctl daemon-reload
sudo systemctl enable docker && sudo systemctl restart docker && sudo systemctl status docker
sudo systemctl daemon-reload
systemctl restart docker
</pre>

#### 5)

#### k8s
#### устанавливаем kubeadm, kubectl, kubelet копирую в этот файл приведенное ниже содержимое.. /etc/yum.repos.d/kubernetes.repo   (add repo)

<pre>
cat <<EOF | > sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
</pre>

#### проверка
<pre>
cat /etc/yum.repos.d/kubernetes.repo
</pre>

#### 5.2)

#### контрольный ребут (смотрим логи)

<pre>
reboot -h now
</pre>

#### 5.3)

#### если нужно установить определенную версию kubernetes смотрим какие версии доступны для установки

<pre>
yum --showduplicates list kubeadm.x86_64
yum --showduplicates list kubeadm.x86_64
yum --showduplicates list kubeadm.x86_64
</pre>

#### примеры инсталяции и деинсталяции

<pre>
yum install -y kubelet-1.26.1-0.x86_64 kubeadm-1.26.1-0.x86_64 kubectl-1.26.1-0.x86_64
yum install -y kubelet-1.26.1-0.x86_64 kubeadm-1.26.1-0.x86_64 kubectl-1.26.1-0.x86_64

yum install -y kubelet-1.24.15-0.x86_64 kubeadm-1.24.15-0.x86_64 kubectl-1.24.15-0.x86_64
yum remove -y kubelet-1.24.15-0.x86_64 kubeadm-1.24.15-0.x86_64 kubectl-1.24.15-0.x86_64

yum install -y kubelet-1.21.0-0.x86_64 kubeadm-1.21.0-0.x86_64 kubectl-1.21.0-0.x86_64
yum install -y kubelet-1.21.0-0.x86_64 kubeadm-1.21.0-0.x86_64 kubectl-1.21.0-0.x86_64

yum install -y kubelet-1.16.2-0.x86_64 kubeadm-1.16.2-0.x86_64 kubectl-1.16.2-0.x86_64
yum install -y kubelet-1.16.2-0.x86_64 kubeadm-1.16.2-0.x86_64 kubectl-1.16.2-0.x86_64
</pre>

#### 5.4)

#### kubelet добавляем в автозагрузку и стартуем

<pre>
sudo systemctl enable --now kubelet && systemctl status kubelet
sudo systemctl enable --now kubelet && systemctl status kubelet
sudo systemctl enable --now kubelet && systemctl status kubelet
</pre>

#### 5.5)

#### смотрим ошибки

<pre>
tail -f /var/log/messages
</pre>


#### ловим ошибку containerd (решаем вопросы) пропускаем этот пункт необрашаем на нее внимания
<pre>
------------------------------
####  пропускаем этот пункт ловим ошибку пропускаем необрашаем внимание:
Jul 13 06:55:55 masterkub kubelet: E0713 06:55:55.831658    1708 run.go:74] "command failed" err="failed to validate kubelet flags: 
the container runtime endpoint address was not specified or empty, use --container-runtime-endpoint to set"

systemctl restart containerd && sudo systemctl enable --now containerd && systemctl status containerd
systemctl restart containerd && sudo systemctl enable --now containerd && systemctl status containerd
systemctl restart containerd && sudo systemctl enable --now containerd && systemctl status containerd

systemctl status containerd
systemctl status containerd
systemctl status containerd

systemctl restart kubelet

### смотрим интерфейсы
ip -4 addr show
ip -4 addr show
ip -4 addr show

rm /etc/containerd/config.toml
rm /etc/containerd/config.toml
rm /etc/containerd/config.toml

systemctl restart containerd && sudo systemctl enable --now containerd && systemctl status containerd
systemctl restart containerd && sudo systemctl enable --now containerd && systemctl status containerd
systemctl restart containerd && sudo systemctl enable --now containerd && systemctl status containerd
------------------------------
</pre>




<pre>
---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
----------------------------------------------настройка patroni или аналог-------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
</pre>

<pre>
мои заметки к диплому выбираем statesfuleset чтобы сохранять измнения контенеров с бд
replicas 3   --- создастца один мастер под и 2 слейва
</pre>


<pre>
---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------ПУНКТЫ КОТОРЫЕ ТРЕБУЕТСЯ ИЗУЧИТЬ!!!!------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
0)
zalando postgres patroni



1)
Local Path Provisioner
Local Path Provisioner
Local Path Provisioner

Диски
Быстродействие любых БД упирается в скорость работы дисковой подсистемы. В kubernetes принято использовать PV и PVC, которые являются не самым быстрым 
по быстродействию решением (за некоторыми исключениями). Быстрые диски - это обычно локальные диски серверов, для доступа к которым используется volumes типа hostPath. 
С другой стороны, как нам говорят гуру ИБ: "использование hostPath не безопасно". Скорее всего из-за этого утверждения в большинстве известных мне helm charts 
и операторах диски определяются только при помощи PVC.
Ещё одна специфика PVC/PV - нет привязки диска к конкретной ноде кластера kubernetes (есть исключения из этого утверждения). 
Т.е. если под переедет на другую ноду кластера, то он сможет подключить существующий PV без создания нового физического диска и копирования (восстановления из бекапа).
Под капотом у PVC/PV в подавляющем большинстве случаев лежит какая-либо сетевая файловая система. Или файловая система с репликацией данных по сети (например longhorn).
В итоге нам предлагают использовать универсальное, но не самое быстрое решение. "Не самое быстрое" - это весьма условно. 
Для многих задач хватит быстродействия PVC/PV. Но есть задачи, где использования PVC/PV будет тормозом. И нам придётся "выкручиваться" подставляя hostPath в чарты (пример minio на hostPath).

Итого:
• В простейшем случае для volumes используем сетевую файловую систему. Например, nfs и nfs-client-provisioner.
• Если хочется использовать локальные диски нод кластера, с репликацией и вменяемой системой управления - смотрим в сторону longhorn.
• Если нужна скорость - тогда используем локальные диски нод напрямую - hostPath или local-path-provisioner.

local-path-provisioner
B local-path-provisioner:v0.0.24 ограничение по объему PV не поддерживается!
Поскольку специалисты ИБ не очень любят hostPath. Сделаем "ход конём", "оденем" локальные диски в PVC при помощи local-path-provisioner.
Небольшое замечание. Раньше, для запрета использования hostPath администраторы кластера могли определять PSP. Но, начиная с kubernetes v1.25.0 PSP переведён
в статус deprecated. :( Я пока не думал как в новых кластерах ввести ограничение на hostPath. Но скорее всего придётся пользоваться внешними инструментами, muna kyverno.
Манифест для установки приложения 00-local-path-storage.yaml.
</pre>







<pre>
---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
----------------------------------------------настройка patroni часть 1 подготовка-----------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
</pre>



1)
установить настроить local-path-provisioner


mkdir /data
mkdir /data/local-path-provisioner
vim /data/00-local-path-storage.yaml


В конфигурации по умолчанию, на всех нодах кластера, если потребуется разместить volume. 
Директория этого тома будет создаваться в /data/local-path-provisioner. Для каждого volume отдельная директория.

--------------------------------------------------------------------------------------
apiVersion: v1
kind: Namespace
metadata:
  name: local-path-storage

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: local-path-provisioner-service-account
  namespace: local-path-storage

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: local-path-provisioner-role
rules:
  - apiGroups: [ "" ]
    resources: [ "nodes", "persistentvolumeclaims", "configmaps" ]
    verbs: [ "get", "list", "watch" ]
  - apiGroups: [ "" ]
    resources: [ "endpoints", "persistentvolumes", "pods" ]
    verbs: [ "*" ]
  - apiGroups: [ "" ]
    resources: [ "events" ]
    verbs: [ "create", "patch" ]
  - apiGroups: [ "storage.k8s.io" ]
    resources: [ "storageclasses" ]
    verbs: [ "get", "list", "watch" ]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: local-path-provisioner-bind
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: local-path-provisioner-role
subjects:
  - kind: ServiceAccount
    name: local-path-provisioner-service-account
    namespace: local-path-storage

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: local-path-provisioner
  namespace: local-path-storage
spec:
  replicas: 1
  selector:
    matchLabels:
      app: local-path-provisioner
  template:
    metadata:
      labels:
        app: local-path-provisioner
    spec:
      serviceAccountName: local-path-provisioner-service-account
      containers:
        - name: local-path-provisioner
          image: rancher/local-path-provisioner:v0.0.24
          imagePullPolicy: IfNotPresent
          command:
            - local-path-provisioner
            - --debug
            - start
            - --config
            - /etc/config/config.json
          volumeMounts:
            - name: config-volume
              mountPath: /etc/config/
          env:
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
      volumes:
        - name: config-volume
          configMap:
            name: local-path-config

---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-path
provisioner: rancher.io/local-path
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Delete

---
kind: ConfigMap
apiVersion: v1
metadata:
  name: local-path-config
  namespace: local-path-storage
data:
  config.json: |-
    {
            "nodePathMap":[
            {
                    "node":"DEFAULT_PATH_FOR_NON_LISTED_NODES",
                    "paths":["/data/local-path-provisioner"]
            }
            ]
    }
  setup: |-
    #!/bin/sh
    set -eu
    mkdir -m 0777 -p "$VOL_DIR"
  teardown: |-
    #!/bin/sh
    set -eu
    rm -rf "$VOL_DIR"
  helperPod.yaml: |-
    apiVersion: v1
    kind: Pod
    metadata:
      name: helper-pod
    spec:
      containers:
      - name: helper-pod
        image: busybox:1.35.0
        imagePullPolicy: IfNotPresent
--------------------------------------------------------------------------------------------------

заметка:
В local-path-provisioner:v0.0.24 ограничение по объему PV не поддерживается!

Установка local-path-provisioner

### посмотрим дефолтный тип стораджа
kubectl get storageclasses
kubectl get storageclasses
kubectl get storageclasses

#если надо удалить
kubectl delete storageclasses local-path
kubectl delete storageclasses local-path
kubectl delete storageclasses local-path

#применяем
kubectl apply -f /data/00-local-path-storage.yaml

#проверяем
kubectl get all -A
------------------------------
NAMESPACE            NAME                                     READY   UP-TO-DATE   AVAILABLE   AGE
kube-system          deployment.apps/coredns                  2/2     2            2           3d4h
local-path-storage   deployment.apps/local-path-provisioner   1/1     1            1           2m33s
-----------------------------

#Создаем PVC где его имя name: volume-test-pvc
В файле my_pvc.yaml показан пример PVC.

vim /data/my_pvc.yaml 

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: volume-test-pvc
  namespace: default
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: local-path
  resources:
    requests:
      storage: 200Mi

Обычно в качестве локальных файловых систем нод кластера используются не кластерные файловые системы. Поэтому в PVC указываем accessModes ReadWriteOnce.
Добавляем PVC в кластер.
kubectl apply -f /data/my_pvc.yaml
Проверяем состояние PVC.
------------------
[root@masterkub data]# kubectl get pvc
NAME              STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
volume-test-pvc   Pending                                      local-path     113s
------------------
status pending говорит нам о том что он не распределяет диск физически до тех пор пока к этому pvc никто не обратится

Т.е. в дальнейшем наше приложение можно будет запускать только на ноде, где создан PV для используемого в приложении PVC.



#черновик он нам понадобится:
---
apiVersion: v1
kind: Pod
metadata:
  name: volume-test
  namespace: default
spec:
  containers:
  - name: volume-test
    image: nginx:1.23.4-alpine
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - name: volv
      mountPath: /data
    ports:
    - containerPort: 80
  nodeSelector:
    kubernetes.io/hostname: ws1.kryukov.local     #вот тут мы указываем где на какой ноде будет создан этот диск
  volumes:
  - name: volv
    persistentVolumeClaim:
      claimName: volume-test-pvc




#Использование в StatefulSet
Как мы поняли из предыдущего примера, при использовании StorageClass связанных с local-path-provisioner мы должны внимательно следить где будут запускаться приложения.

Самым правильным вариантом, когда нам требуется сохранять состояния приложений в файловой системе, является использование StatefulSet.

При определении StatefulSet мы должны будем учесть следующие особенности:

1)Явным образом определить ноды, на которых будут запускаться pods StatefulSet-та.
2)Позаботиться о том, что бы на одной ноде запускался один pod StatefulSet-та.




<pre>
---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
----------------------------------------------настройка patroni часть 2 настройка базового yaml----------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
</pre>

Преходим к следуюшему шагу подкготовки базоввому yaml для patroni от zalando/spilo (будем использовать контенеры для построения кластера patroni в kub)



#### добавил метки они нам нужны чтобы разкатать patroni по нашим указанным нодам

<pre>
kubectl label nodes kub1.ru-central1.internal db=spilo
kubectl label nodes kub2.ru-central1.internal db=spilo
kubectl label nodes kub3.ru-central1.internal db=spilo

kubectl get nodes --show-labels

root@masterkub vorori]# kubectl get nodes --show-labels
NAME                             STATUS   ROLES           AGE     VERSION   LABELS
kub1.ru-central1.internal        Ready    <none>          3d21h   v1.26.1   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,db=spilo,
kub2.ru-central1.internal        Ready    <none>          3d21h   v1.26.1   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,db=spilo,
kub3.ru-central1.internal        Ready    <none>          3d21h   v1.26.1   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,db=spilo,
masterkub.ru-central1.internal   Ready    control-plane   3d22h   v1.26.1   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,
</pre>


---------------------------------------------------------------------------------------------------------------------------------------
----------------------------------------------Конфигурационные параметры скрипта spilo_backup START-------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------

============================================================================================
============================================================================================
============================================================================================
#создаем окружения для  скрипта backup
--------------------------
rm -rf /data/spilo_backup-script_suka.yaml
vim /data/spilo_backup-script_suka.yaml
cat /data/spilo_backup-script_suka.yaml
--------------------------

#применяю
--------------------------
kubectl apply -f /data/spilo_backup-script_suka.yaml
kubectl apply -f /data/spilo_backup-script_suka.yaml
--------------------------

#проверяем
--------------------------
kubectl get ConfigMap
kubectl get ConfigMap
--------------------------

#если надо удалить
--------------------------
kubectl delete configmap backup-script
kubectl delete configmap backup-script
--------------------------

--------------------------
--------------------------
--------------------------
--------------------------
apiVersion: v1
kind: ConfigMap
metadata:
  name: backup-script
data:
  PGHOST: "/var/run/postgresql"
  PGUSER: "postgres"
  PGROOT: "/home/postgres/pgdata/pgroot"
  PGLOG: "/home/postgres/pgdata/pgroot/pg_log"
  PGDATA: "/home/postgres/pgdata/pgroot/data"
  BACKUP_NUM_TO_RETAIN: "5"
  USE_WALG_BACKUP: "true"
  USE_WALG_RESTORE: "true"
  WAL_BUCKET_SCOPE_PREFIX: ""
  WAL_BUCKET_SCOPE_SUFFIX: ""
  WALG_ALIVE_CHECK_INTERVAL: "5m"
  WALE_BINARY: "wal-g"
  WALG_FILE_PREFIX: "/data/pg_wal"
  CLONE_USE_WALG_RESTORE: "true"
  WALG_DISABLE_S3_SSE: "true"
  WALE_ENV_DIR: "/config
--------------------------
--------------------------
--------------------------
--------------------------


============================================================================================
============================================================================================
============================================================================================


---------------------------------------------------------------------------------------------------------------------------------------
----------------------------------------------Конфигурационные параметры скрипта spilo_backup END---------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------




---------------------------------------------------------------------------------------------------------------------------------------
----------------------------------------------Конфигурационные параметры zalandopatroni final START------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------

============================================================================================
============================================================================================
============================================================================================
#начинаем создание нашего кластера

#создаем окружения для  основной конфигурации
-------------------------- 
rm -rf /data/spilo_kubernetes_final_suka.yaml
vim /data/spilo_kubernetes_final_suka.yaml
cat /data/spilo_kubernetes_final_suka.yaml
--------------------------


#создаем namespase spilo
--------------------------
kubectl create ns spilo
kubectl create ns spilo
--------------------------

#применяю
--------------------------
kubectl apply -f /data/spilo_kubernetes_final_suka.yaml --namespace spilo
kubectl apply -f /data/spilo_kubernetes_final_suka.yaml --namespace spilo
--------------------------

#проверяю
--------------------------
# pods
kubectl get pods --namespace spilo
kubectl get pods --namespace spilo

# pwc
kubectl get pvc --namespace spilo
kubectl get pvc --namespace spilo

# Это предоставит информацию о пространствах имен
kubectl get namespace
kubectl get namespace
--------------------------


#если надо удалить
--------------------------
#если надо удалить манифест
kubectl delete -f /data/spilo_kubernetes_final_suka.yaml  --namespace spilo
kubectl delete -f /data/spilo_kubernetes_final_suka.yaml  --namespace spilo

#если надо удалить  all,ing,secrets,pvc,pv
kubectl delete all,ing,secrets,pvc,pv --all --namespace spilo
kubectl delete all,ing,secrets,pvc,pv --all --namespace spilo

#после удаляю руками persistentvolume которые не удалились автоматом  из директории  
rm -rf /data/local-path-provisioner/*
rm -rf /data/local-path-provisioner/*
rm -rf /data/local-path-provisioner/*

# дополнительно если надо удалить  namespace spilo
kubectl delete namespace spilo
kubectl delete namespace spilo
--------------------------


--------------------------
--------------------------
--------------------------
--------------------------
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: &cluster_name zalandopatroni777
  labels:
    application: spilo
    spilo-cluster: *cluster_name
spec:
  selector:
    matchLabels:
      application: spilo
      spilo-cluster: *cluster_name
  replicas: 3
  serviceName: *cluster_name
  podManagementPolicy: Parallel
  template:
    metadata:
      labels:
        application: spilo
        spilo-cluster: *cluster_name
    spec:
      # service account that allows changing endpoints and assigning pod labels
      # in the given namespace: https://kubernetes.io/docs/user-guide/service-accounts/
      # not required unless you've changed the default service account in the namespace
      # used to deploy Spilo
      serviceAccountName: operator
      containers:
      - name: *cluster_name
        image: registry.opensource.zalan.do/acid/spilo-15:3.0-p1  # put the spilo image here
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8008
          protocol: TCP
        - containerPort: 5432
          protocol: TCP
        volumeMounts:
        - mountPath: /data/pg_wal
          name: backup
        - mountPath: /config
          name: config
        - mountPath: /home/postgres/pgdata
          name: pgdata
        env:
        - name: DCS_ENABLE_KUBERNETES_API
          value: 'true'
#        - name: ETCD_HOST
#          value: 'test-etcd.default.svc.cluster.local:2379' # where is your etcd?
#        - name: WAL_S3_BUCKET
#          value: example-spilo-dbaas
#        - name: LOG_S3_BUCKET # may be the same as WAL_S3_BUCKET
#          value: example-spilo-dbaas
#        - name: BACKUP_SCHEDULE
#          value: "00 01 * * *"
        - name: KUBERNETES_SCOPE_LABEL
          value: spilo-cluster
        - name: KUBERNETES_ROLE_LABEL
          value: role
        - name: SPILO_CONFIGURATION
          value: | ## https://github.com/zalando/patroni#yaml-configuration
            bootstrap:
              dcs:
                ttl: 100
                loop_wait: 10
                retry_timeout: 10
                maximum_lag_on_failover: 1048576
                master_start_timeout: 300
                synchronous_mode_strict: false
                postgresql:
                  use_pg_rewind: true
                  use_slots: true
                  parameters:
                    max_connections: 101
                    shared_buffers : 2GB
                    effective_cache_size : 7GB
                    maintenance_work_mem: 512MB
                    wal_buffers: 16MB
                    wal_keep_size: 2GB
                    work_mem: 32MB
                    min_wal_size: 1GB
                    max_wal_size: 4GB
                    huge_pages: off
                    max_worker_processes: 4
                    max_parallel_workers: 4
                    max_parallel_workers_per_gather: 2
                    max_parallel_maintenance_workers: 2
                    autovacuum: on
                    autovacuum_max_workers: 4
                    autovacuum_vacuum_scale_factor: 0.01
                    autovacuum_analyze_scale_factor: 0.03
                    autovacuum_vacuum_cost_limit: 500
                    autovacuum_vacuum_cost_delay: 2
                    autovacuum_naptime: 15s
                    autovacuum_vacuum_threshold: 20
                    wal_writer_delay : 200ms
                    wal_writer_flush_after : 1MB
                    random_page_cost: 1.1
                    seq_page_cost: 1
                    effective_io_concurrency: 200
                    superuser_reserved_connections: 4
                    max_locks_per_transaction: 64
                    max_prepared_transactions: 0
                    checkpoint_timeout: 10min
                    checkpoint_completion_target: 0.9
                    default_statistics_target: 1000
                    synchronous_commit: on
                    max_files_per_process: 1024
                    wal_level: replica
                    max_wal_senders: 10
                    max_replication_slots: 10
                    hot_standby: on
                    wal_compression: on
                    track_io_timing: on
                    log_lock_waits: on
                    log_temp_files: 0
                    track_activities: on
                    track_counts: on
                    track_functions: all
                    log_checkpoints: off
                    log_connections: off
                    log_disconnections: off
                    log_statement: none
                    logging_collector: on
                    log_min_duration_statement: 30s
                    log_truncate_on_rotation: on
                    log_rotation_age: 1d
                    log_rotation_size: 0
                    log_line_prefix: '%m [%p] %d %u %h (transaction ID %x)'
                    max_standby_streaming_delay: 30s
                    wal_receiver_status_interval: 10s
                    jit: off
                    lc_messages: en_US.UTF-8
                    track_commit_timestamp: "off"
                    wal_log_hints: on
                    hot_standby_feedback: off
                  pg_hba:
                  - hostnossl all     vororinew      all          md5
              initdb:
                - encoding: UTF8
                - data-checksums
              pg_hba:
              - hostnossl all         vorori         all          md5
            postgresql:
                parameters:
                  log_destination: 'stderr'
                  log_line_prefix: '%m [%p] %d %u %h (transaction ID %x)'
                  archive_mode: on
                  archive_command: 'envdir /config /usr/local/bin/wal-g wal-push "%p"'
                recovery_conf:
                  restore_command: 'envdir /config /usr/local/bin/wal-g wal-fetch "%f" "%p"'
                pg_hba:
                - local     all         all                    trust
                - hostssl   all         +zalandos 127.0.0.1/32 pam
                - host      all         all       127.0.0.1/32 md5
                - hostssl   all         +zalandos ::1/128      pam
                - host      all         all       ::1/128      md5
                - local     replication standby                trust
                - hostssl   replication standby   all          md5
                - hostnossl all         all       all          md5
                - hostssl   all         +zalandos all          pam
                - hostssl   all         all       all          md5
        - name: POD_IP
          valueFrom:
            fieldRef:
              apiVersion: v1
              fieldPath: status.podIP
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
        - name: PGPASSWORD_SUPERUSER
          valueFrom:
            secretKeyRef:
              name: *cluster_name
              key: superuser-password
        - name: PGUSER_ADMIN
          value: superadmin
        - name: PGPASSWORD_ADMIN
          valueFrom:
            secretKeyRef:
              name: *cluster_name
              key: admin-password
        - name: PGPASSWORD_STANDBY
          valueFrom:
            secretKeyRef:
              name: *cluster_name
              key: replication-password
        - name: SCOPE
          value: *cluster_name
        - name: PGROOT
          value: /home/postgres/pgdata/pgroot
        - name: WALG_FILE_PREFIX
          value: "/home/postgres/pgdata/pgroot/pg_log"
        - name: CRONTAB
          value: "[\"00 01 * * * envdir /config /scripts/postgres_backup.sh /home/postgres/pgdata/pgroot/data\"]"
      terminationGracePeriodSeconds: 0
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: db
                    operator: In
                    values:
                      - spilo
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: spilo-cluster
                    operator: In
                    values:
                      - *cluster_name
              topologyKey: "kubernetes.io/hostname"
      volumes:
        - configMap:
            name: backup-script
          name: config
        - persistentVolumeClaim:
            claimName: zalandopatroni777-backup
          name: backup
  volumeClaimTemplates:
  - metadata:
      labels:
        application: spilo
        spilo-cluster: *cluster_name
      name: pgdata
    spec:
      storageClassName: local-path
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 10Gi
  - metadata:
      labels:
        application: spilo
        spilo-cluster: *cluster_name
      name: backup
    spec:
      storageClassName: local-path
      accessModes:
       - ReadWriteOnce
      resources:
        requests:
          storage: 3Gi
---
apiVersion: v1
kind: Endpoints
metadata:
  name: &cluster_name zalandopatroni777
  labels:
    application: spilo
    spilo-cluster: *cluster_name
subsets: []

---
apiVersion: v1
kind: Service
metadata:
  name: &cluster_name zalandopatroni777
  labels:
    application: spilo
    spilo-cluster: *cluster_name
spec:
  type: ClusterIP
  ports:
  - name: postgresql
    port: 5432
    targetPort: 5432

---
# headless service to avoid deletion of patronidemo-config endpoint
apiVersion: v1
kind: Service
metadata:
  name: zalandopatroni777-config
  labels:
    application: spilo
    spilo-cluster: zalandopatroni777
spec:
  clusterIP: None

---
apiVersion: v1
kind: Secret
metadata:
  name: &cluster_name zalandopatroni777
  labels:
    application: spilo
    spilo-cluster: *cluster_name
type: Opaque
stringData:
  superuser-password: pass1
  replication-password: pass2
  admin-password: pass3

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: operator

---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: operator
rules:
- apiGroups:
  - ""
  resources:
  - configmaps
  verbs:
  - create
  - get
  - list
  - patch
  - update
  - watch
  # delete is required only for 'patronictl remove'
  - delete
- apiGroups:
  - ""
  resources:
  - endpoints
  verbs:
  - get
  - patch
  - update
  # the following three privileges are necessary only when using endpoints
  - create
  - list
  - watch
  # delete is required only for for 'patronictl remove'
  - delete
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - get
  - list
  - patch
  - update
  - watch
# The following privilege is only necessary for creation of headless service
# for patronidemo-config endpoint, in order to prevent cleaning it up by the
# k8s master. You can avoid giving this privilege by explicitly creating the
# service like it is done in this manifest (lines 160..169)
- apiGroups:
  - ""
  resources:
  - services
  verbs:
  - create

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: operator
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: operator
subjects:
- kind: ServiceAccount
  name: operator
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: backup-script
data:
  PGHOST: "/var/run/postgresql"
  PGUSER: "postgres"
  PGROOT: "/home/postgres/pgdata/pgroot"
  PGLOG: "/home/postgres/pgdata/pgroot/pg_log"
  PGDATA: "/home/postgres/pgdata/pgroot/data"
  BACKUP_NUM_TO_RETAIN: "5"
  USE_WALG_BACKUP: "true"
  USE_WALG_RESTORE: "true"
  WAL_BUCKET_SCOPE_PREFIX: ""
  WAL_BUCKET_SCOPE_SUFFIX: ""
  WALG_ALIVE_CHECK_INTERVAL: "5m"
  WALE_BINARY: "wal-g"
  WALG_FILE_PREFIX: "/data/pg_wal"
  CLONE_USE_WALG_RESTORE: "true"
  WALG_DISABLE_S3_SSE: "true"
  WALE_ENV_DIR: "/config"
--------------------------
--------------------------
--------------------------
--------------------------


---------------------------------------------------------------------------------------------------------------------------------------
----------------------------------------------Конфигурационные параметры zalandopatroni final END--------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------


#проверяю что все ок и что все запустилось все под создались и работают
kubectl get pods --namespace spilo
kubectl get pods --namespace spilo

----------
kubectl get pods --namespace spilo
NAME                  READY   STATUS    RESTARTS   AGE
zalandopatroni777-0   1/1     Running   0          2m22s
zalandopatroni777-1   1/1     Running   0          2m22s
zalandopatroni777-2   1/1     Running   0          2m22s
----------

#проверяю что создались сервисы
kubectl get services --namespace spilo
kubectl get services --namespace spilo

----------
NAME                      TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
zalandopatroni777          ClusterIP   10.99.158.120   <none>        5432/TCP   2m58s
zalandopatroni777-config   ClusterIP   None            <none>        <none>     2m58s
----------

#проверяю что все ок и что все запустилось с дисками pvc
kubectl get pvc --namespace spilo
kubectl get pvc --namespace spilo

----------
backup-zalandopatroni777-0   Bound    pvc-7998c73c-af30-495a-936b-d282c753968e   3Gi        RWO            local-path     3m18s
backup-zalandopatroni777-1   Bound    pvc-e9977701-028c-4ebf-b674-ac4ab9f28f4a   3Gi        RWO            local-path     3m17s
backup-zalandopatroni777-2   Bound    pvc-03a38737-e58d-4d2f-90f4-f265c4a089f8   3Gi        RWO            local-path     3m17s
pgdata-zalandopatroni777-0   Bound    pvc-501f1dc7-9bde-4ddd-8624-f5e69832d0ab   10Gi       RWO            local-path     3m18s
pgdata-zalandopatroni777-1   Bound    pvc-8988e08d-1145-4491-a354-0a6e35f44601   10Gi       RWO            local-path     3m17s
pgdata-zalandopatroni777-2   Bound    pvc-32147774-ce25-4dcf-ab67-1edbbe57cb9a   10Gi       RWO            local-path     3m17s
----------

#проверяю что все ок на всех обектах кластера
kubectl get all -A
----------
NAMESPACE            NAME                                                         READY   STATUS    RESTARTS       AGE
kube-flannel         pod/kube-flannel-ds-77pwz                                    1/1     Running   11 (68m ago)   4d23h
kube-flannel         pod/kube-flannel-ds-8nrhx                                    1/1     Running   8 (67m ago)    4d23h
kube-flannel         pod/kube-flannel-ds-d94qd                                    1/1     Running   8 (67m ago)    4d23h
kube-flannel         pod/kube-flannel-ds-pslvk                                    1/1     Running   7 (68m ago)    4d23h
kube-system          pod/coredns-787d4945fb-mxvx7                                 1/1     Running   7 (68m ago)    5d
kube-system          pod/coredns-787d4945fb-x57tb                                 1/1     Running   7 (68m ago)    5d
kube-system          pod/etcd-masterkub.ru-central1.internal                      1/1     Running   7 (68m ago)    5d
kube-system          pod/kube-apiserver-masterkub.ru-central1.internal            1/1     Running   7 (68m ago)    5d
kube-system          pod/kube-controller-manager-masterkub.ru-central1.internal   1/1     Running   7 (68m ago)    5d
kube-system          pod/kube-proxy-25mpq                                         1/1     Running   8 (67m ago)    4d23h
kube-system          pod/kube-proxy-cdhx7                                         1/1     Running   7 (68m ago)    5d
kube-system          pod/kube-proxy-ssz9x                                         1/1     Running   8 (68m ago)    4d23h
kube-system          pod/kube-proxy-z4prx                                         1/1     Running   7 (68m ago)    4d23h
kube-system          pod/kube-scheduler-masterkub.ru-central1.internal            1/1     Running   7 (68m ago)    5d
local-path-storage   pod/local-path-provisioner-7f8667b75c-swvwb                  1/1     Running   7 (68m ago)    43h
spilo                pod/zalandopatroni777-0                                      1/1     Running   0              3m37s
spilo                pod/zalandopatroni777-1                                      1/1     Running   0              3m37s
spilo                pod/zalandopatroni777-2                                      1/1     Running   0              3m37s

NAMESPACE     NAME                               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
default       service/kubernetes                 ClusterIP   10.96.0.1       <none>        443/TCP                  101m
kube-system   service/kube-dns                   ClusterIP   10.96.0.10      <none>        53/UDP,53/TCP,9153/TCP   5d
spilo         service/zalandopatroni777          ClusterIP   10.109.75.112   <none>        5432/TCP                 3m38s
spilo         service/zalandopatroni777-config   ClusterIP   None            <none>        <none>                   3m38s

NAMESPACE      NAME                             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-flannel   daemonset.apps/kube-flannel-ds   4         4         4       4            4           <none>                   4d23h
kube-system    daemonset.apps/kube-proxy        4         4         4       4            4           kubernetes.io/os=linux   5d

NAMESPACE            NAME                                     READY   UP-TO-DATE   AVAILABLE   AGE
kube-system          deployment.apps/coredns                  2/2     2            2           5d
local-path-storage   deployment.apps/local-path-provisioner   1/1     1            1           43h

NAMESPACE            NAME                                                DESIRED   CURRENT   READY   AGE
kube-system          replicaset.apps/coredns-787d4945fb                  2         2         2       5d
local-path-storage   replicaset.apps/local-path-provisioner-7f8667b75c   1         1         1       43h

NAMESPACE   NAME                                 READY   AGE
spilo       statefulset.apps/zalandopatroni777   3/3     3m38s
----------



#смотрим на кворум кто мастер из самого пода pod/zalandopatroni01-0 
kubectl -n spilo exec -it pod/zalandopatroni777-0  -- patronictl list
kubectl -n spilo exec -it pod/zalandopatroni777-0  -- patronictl list

----------
+ Cluster: zalandopatroni777 -------+---------+---------+----+-----------+
| Member              | Host        | Role    | State   | TL | Lag in MB |
+---------------------+-------------+---------+---------+----+-----------+
| zalandopatroni777-0 | 10.244.2.31 | Leader  | running |  1 |           |
| zalandopatroni777-1 | 10.244.1.33 | Replica | running |  1 |         0 |
| zalandopatroni777-2 | 10.244.3.36 | Replica | running |  1 |         0 |
+---------------------+-------------+---------+---------+----+-----------+
----------

### единоразово смотрим логи pod видм кто у нас матер
kubectl logs --namespace spilo pod/zalandopatroni777-0
kubectl logs --namespace spilo pod/zalandopatroni777-1
kubectl logs --namespace spilo pod/zalandopatroni777-2

### смотреть логи в режиме реального времени pod pod/zalandopatroni01-0 namespace spilo используя префикс -f
kubectl logs --namespace spilo pod/zalandopatroni777-0 -f
kubectl logs --namespace spilo pod/zalandopatroni777-0 -f
----------
2023-08-16 08:48:22,622 INFO: no action. I am (zalandopatroni777-0), the leader with the lock
2023-08-16 08:48:32,625 INFO: no action. I am (zalandopatroni777-0), the leader with the lock
2023-08-16 08:48:42,636 INFO: no action. I am (zalandopatroni777-0), the leader with the lock

2023-08-16 08:48:52,640 INFO: no action. I am (zalandopatroni777-1), a secondary, and following a leader (zalandopatroni777-0)
2023-08-16 08:49:02,635 INFO: no action. I am (zalandopatroni777-1), a secondary, and following a leader (zalandopatroni777-0)
2023-08-16 08:49:12,637 INFO: no action. I am (zalandopatroni777-1), a secondary, and following a leader (zalandopatroni777-0)

2023-08-16 08:49:02,629 INFO: no action. I am (zalandopatroni777-2), a secondary, and following a leader (zalandopatroni777-0)
2023-08-16 08:49:12,629 INFO: no action. I am (zalandopatroni777-2), a secondary, and following a leader (zalandopatroni777-0)
2023-08-16 08:49:22,630 INFO: no action. I am (zalandopatroni777-2), a secondary, and following a leader (zalandopatroni777-0)
----------



#устанавливаю клиента psql на сервер чтобы подключиться к сервису
yum install postgresql
yum install postgresql

#пытаемся подключиться получаем ошибку это значинт надо моменять конфиг pg_hba.conf
psql -U postgres -h 10.99.158.120
psql -U postgres -h 10.99.158.120

#заметка:
пароль для postgres в нашем случае что указан в конфигурационном yaml: pass1
пароль для postgres в нашем случае что указан в конфигурационном yaml: pass1

#Наборы скриптов для patroni находятся тут
kubectl -n spilo exec -it pod/zalandopatroni777-0 -- ls /scripts/
kubectl -n spilo exec -it pod/zalandopatroni777-0 -- ls /scripts/

kubectl -n spilo exec -it pod/zalandopatroni777-0 -- cat /scripts/basebackup.sh
kubectl -n spilo exec -it pod/zalandopatroni777-0 -- cat /scripts/basebackup.sh

#запустить скрипт бекапа
kubectl -n spilo exec -it pod/zalandopatroni777-0 -- bash /scripts/basebackup.sh
kubectl -n spilo exec -it pod/zalandopatroni777-0 -- bash /scripts/basebackup.sh


#если требуется изменить конфигурацию кластера zalandopatroni01
kubectl -n spilo exec -it pod/zalandopatroni777-0  -- patronictl restart zalandopatroni777
kubectl -n spilo exec -it pod/zalandopatroni777-0  -- patronictl restart zalandopatroni777

kubectl -n spilo exec -it pod/zalandopatroni777-0  -- patronictl reload zalandopatroni777
kubectl -n spilo exec -it pod/zalandopatroni777-0  -- patronictl reload zalandopatroni777


#подключаемся к лидеру напрямую внутрь контенера с patroni
kubectl -n spilo exec -it pod/zalandopatroni777-0  -- bash
kubectl -n spilo exec -it pod/zalandopatroni777-0  -- bash

---------------
 ___        _ _
/ ___| _ __ (_) | ___
\___ \| '_ \| | |/ _ \
 ___) | |_) | | | (_) |
|____/| .__/|_|_|\___/
      |_|

This container is managed by runit, when stopping/starting services use sv

Examples:

sv stop cron
sv restart patroni

Current status: (sv status /etc/service/*)

run: /etc/service/cron: (pid 37) 924s
run: /etc/service/patroni: (pid 38) 924s
run: /etc/service/pgqd: (pid 36) 924s
---------------

#нашли файл с конфигурацией patroni
pwd
---------------
/home/postgres
---------------
ls -l
---------------
total 4
lrwxrwxrwx. 1 root     root   8 Mar 10 05:53 etc -> /run/etc
drwxrwxrwx. 3 root     root  20 Aug 16 08:42 pgdata
-rw-rw-r--. 1 postgres root 157 Mar 10 05:37 pgq_ticker.ini
lrwxrwxrwx. 1 root     root  17 Mar 10 05:53 postgres.yml -> /run/postgres.yml
---------------

#редактируем свои настройки из внутрянки пода 
-----------------------------------------------------------------------------
#команда на изменение параметра . ниже показан как я добавил раздел pg_hba::
patronictl -c postgres.yml edit-config
patronictl -c postgres.yml edit-config
patronictl -c postgres.yml edit-config
---------------------

#если требуется restart
patronictl -c postgres.yml restart zalandopatroni777
patronictl -c postgres.yml restart zalandopatroni777

#если требуется reload
patronictl -c postgres.yml reload zalandopatroni777
patronictl -c postgres.yml reload zalandopatroni777

-----------------------------------------------------------------------------
#показать лидера и кворум из внутрянки пода 
-----------------------------------------------------------------------------
показать лидера и кворум из внутрянки пода 
patronictl -c postgres.yml list
patronictl -c postgres.yml list
patronictl -c postgres.yml list
-----------------------------------------------------------------------------



#БАЗОВАЯ ИНФОРМАЦИЯ порт версия сервер
select
inet_server_addr( ) AS "Server",
inet_server_port( ) AS "Port",
current_database() AS "CurrentDatabase",
version() AS "Version";

--------------------------
   Server    | Port | CurrentDatabase |                                                              Version
-------------+------+-----------------+-----------------------------------------------------------------------------------------------------------------------------------
 10.244.1.37 | 5432 | postgres        | PostgreSQL 15.2 (Ubuntu 15.2-1.pgdg22.04+1) on x86_64-pc-linux-gnu, compiled by gcc (Ubuntu 11.3.0-1ubuntu1~22.04) 11.3.0, 64-bit
(1 row)
--------------------------

#показать ноды реплики
select usename,application_name,client_addr,backend_start,state,sync_state from pg_stat_replication;

--------------------------
postgres=# select usename,application_name,client_addr,backend_start,state,sync_state from pg_stat_replication;
 usename |  application_name   | client_addr |         backend_start         |   state   | sync_state
---------+---------------------+-------------+-------------------------------+-----------+------------
 standby | zalandopatroni777-1 | 10.244.2.36 | 2023-08-16 11:01:20.669813+00 | streaming | async
 standby | zalandopatroni777-2 | 10.244.3.40 | 2023-08-16 11:01:21.942592+00 | streaming | async
(2 rows)
--------------------------

#показать состояние конкретной ноды к которой подключился не находится ли она в состоянии восстоновления
select pg_is_in_recovery();

#выполняю преключение
kubectl -n spilo exec -it pod/zalandopatroni777-1 -- patronictl list

+ Cluster: zalandopatroni777 -------+---------+---------+----+-----------+
| Member              | Host        | Role    | State   | TL | Lag in MB |
+---------------------+-------------+---------+---------+----+-----------+
| zalandopatroni777-0 | 10.244.1.37 | Leader  | running |  1 |           |
| zalandopatroni777-1 | 10.244.2.36 | Replica | running |  1 |         0 |
| zalandopatroni777-2 | 10.244.3.40 | Replica | running |  1 |         0 |
+---------------------+-------------+---------+---------+----+-----------+

#выполняю switchover
kubectl -n spilo exec -it pod/zalandopatroni777-1  -- patronictl switchover
kubectl -n spilo exec -it pod/zalandopatroni777-1  -- patronictl switchover

#видим лидер изменился и TL
+ Cluster: zalandopatroni777 -------+---------+---------+----+-----------+
| Member              | Host        | Role    | State   | TL | Lag in MB |
+---------------------+-------------+---------+---------+----+-----------+
| zalandopatroni777-0 | 10.244.1.37 | Replica  | running |  1 |           |
| zalandopatroni777-1 | 10.244.2.36 | Leader   | running |  1 |         0 |
| zalandopatroni777-2 | 10.244.3.40 | Replica  | running |  1 |         0 |
+---------------------+-------------+---------+---------+----+-----------+

#создаю аварийную ситуацию роняю под лидера на котором работает кластер патрони прегружаю vm kub1

#на мастере
shutdown -r now
shutdown -r now

#смотрим что под zalandopatroni777-2  стал лидером а zalandopatroni777-1  
#Replica и с лагом репликации 1 и TL не преключился на 3
+ Cluster: zalandopatroni777 -------+---------+---------+----+-----------+
| Member              | Host        | Role    | State   | TL | Lag in MB |
+---------------------+-------------+---------+---------+----+-----------+
| zalandopatroni777-0 | 10.244.1.37 | Replica  | running |  2 |        1 |
| zalandopatroni777-1 | 10.244.2.36 | Replica  | running |  3 |        0 |
| zalandopatroni777-2 | 10.244.3.40 | Leader   | running |  3 |        0 |
+---------------------+-------------+---------+---------+----+-----------+

# делаем принудительную синхронизацию с мастера для пода pod/zalandopatroni777-0
# можно выполнить специальную команду которая пресоздаст нашу реплику с нуля
# мы же попробуем просто убить под с этой репликой и посмотрим на работу kubernets
kubectl logs --namespace spilo pod/zalandopatroni777-0
kubectl logs --namespace spilo pod/zalandopatroni777-1

### единоразово смотрим логи pod видм кто у нас матер
kubectl logs --namespace spilo pod/zalandopatroni777-0 --tail=40 
kubectl logs --namespace spilo pod/zalandopatroni777-0 --tail=40 

-------------------
2023-08-15 10:44:32,059 ERROR: Exception when working with leader
Traceback (most recent call last):
  File "/usr/local/lib/python3.10/dist-packages/patroni/postgresql/rewind.py", line 70, in check_leader_is_not_in_recovery
    with get_connection_cursor(connect_timeout=3, options='-c statement_timeout=2000', **conn_kwargs) as cur:
  File "/usr/lib/python3.10/contextlib.py", line 135, in __enter__
    return next(self.gen)
  File "/usr/local/lib/python3.10/dist-packages/patroni/postgresql/connection.py", line 43, in get_connection_cursor
    conn = psycopg.connect(**kwargs)
  File "/usr/local/lib/python3.10/dist-packages/patroni/psycopg.py", line 42, in connect
    ret = _connect(*args, **kwargs)
  File "/usr/lib/python3/dist-packages/psycopg2/__init__.py", line 122, in connect
    conn = _connect(dsn, connection_factory=connection_factory, **kwasync)
-------------------


#удаляем проблемный под
kubectl delete pod zalandopatroni777-0 --namespace spilo
kubectl delete pod zalandopatroni777-0 --namespace spilo

#проверяем под стартанул 95s
NAME                 READY   STATUS    RESTARTS   AGE
zalandopatroni777-0   1/1     Running   0          95s
zalandopatroni777-1   1/1     Running   0          3h28m
zalandopatroni777-2   1/1     Running   0          3h28m

### смотреть логи в режиме реального времени pod pod/zalandopatroni01-0 namespace spilo используя префикс -f
kubectl logs --namespace spilo pod/zalandopatroni777-0 -f
kubectl logs --namespace spilo pod/zalandopatroni777-0 -f

###умышленнос сломал zalandopatroni777-0 данные повреждены (удалил пару файлов с каталога data)
Traceback (most recent call last):
  File "/usr/local/lib/python3.10/dist-packages/patroni/postgresql/rewind.py", line 70, in check_leader_is_not_in_recovery
    with get_connection_cursor(connect_timeout=3, options='-c statement_timeout=2000', **conn_kwargs) as cur:
  File "/usr/lib/python3.10/contextlib.py", line 135, in __enter__
    return next(self.gen)
  File "/usr/local/lib/python3.10/dist-packages/patroni/postgresql/connection.py", line 43, in get_connection_cursor
    conn = psycopg.connect(**kwargs)
  File "/usr/local/lib/python3.10/dist-packages/patroni/psycopg.py", line 42, in connect
    ret = _connect(*args, **kwargs)
  File "/usr/lib/python3/dist-packages/psycopg2/__init__.py", line 122, in connect
    conn = _connect(dsn, connection_factory=connection_factory, **kwasync)


#наблюдаем вот такую картину start failed Lag in MB unknown
+ Cluster: zalandopatroni777 ------+---------+--------------+----+-----------+
| Member             | Host       | Role    | State        | TL | Lag in MB |
+--------------------+------------+---------+--------------+----+-----------+
| zalandopatroni777-0| 10.244.1.37 | Replica | start failed |    |   unknown|
| zalandopatroni777-1| 10.244.2.36 | Replica | running      |  3 |         0|
| zalandopatroni777-2| 10.244.3.40 | Leader  | running      |  3 |          |
+--------------------+------------+---------+--------------+----+-----------+


#чтобы вернуть оживить под zalandopatroni777-0  я пошел на сервер kub1 и почистил каталог с данными
# после чего patroni пресоздал реплику zalandopatroni777-0
# и мы видим что с кластером все ок  zalandopatroni777-0 реплицировался TL = 3
+ Cluster: zalandopatroni01 ------+---------+---------+----+-----------+
| Member             | Host       | Role    | State   | TL | Lag in MB |
+--------------------+------------+---------+---------+----+-----------+
| zalandopatroni777-0| 10.244.1.37 | Replica | running |  3 |         0|
| zalandopatroni777-1| 10.244.2.36 | Replica | running |  3 |         0|
| zalandopatroni777-2| 10.244.3.40 | Leader  | running |  3 |          |
+--------------------+------------+---------+---------+----+-----------+



#установил на управляюшей ноде postgressudo но кластер не инициализировал мне нужна програмка pg_basebackup 
yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm && yum -y repolist && yum -y install postgresql15-contrib

#выполняю бекап postgres кластера который в кубере 
mkdir /var/postgre_backup
chown postgres:postgres /var/postgre_backup

kubectl get services --namespace spilo
kubectl -n spilo exec -it pod/zalandopatroni777-0  -- bash

#команда на изменение параметра . ниже показан как я добавил раздел pg_hba:: (добавил в pg_hba.conf replication connection from host "10.244.0.0", user "postgres")
patronictl -c postgres.yml edit-config
patronictl -c postgres.yml edit-config

--------------------------------------
loop_wait: 10
master_start_timeout: 300
maximum_lag_on_failover: 1048576
postgresql:
  parameters:
    archive_mode: 'on'
    archive_timeout: 1800s
    autovacuum: true
	wal_buffers: 16MB
    wal_compression: true
    wal_keep_size: 2GB
    wal_level: replica
    wal_log_hints: true
    wal_receiver_status_interval: 10s
    wal_writer_delay: 200ms
    wal_writer_flush_after: 1MB
    work_mem: 32MB
  pg_hba:
  - host replication postgres 10.244.0.0/0 md5
  - host replication all 10.244.0.0/0 md5
  use_pg_rewind: true
  use_slots: true
retry_timeout: 10
synchronous_mode_strict: false
ttl: 100
--------------------------------------


#если надо выполнить через 
su - postgres -c "/usr/pgsql-15/bin/pg_basebackup -U postgres -p 5432 -h 10.99.158.120 --progress --verbose --format=tar --gzip --pgdata=/var/postgre_backup/all_db_postgre_backup_`date +"%Y_%m_%d_%H:%M"`"
su - postgres -c "/usr/pgsql-15/bin/pg_basebackup -U postgres -p 5432 -h 10.99.158.120 --progress --verbose --format=tar --gzip --pgdata=/var/postgre_backup/all_db_postgre_backup_`date +"%Y_%m_%d_%H:%M"`"

#выполним backup
envdir /config /scripts/postgres_backup.sh /home/postgres/pgdata/pgroot/data
envdir /config /scripts/postgres_backup.sh /home/postgres/pgdata/pgroot/data

--------------------------------------
postgres@zalandopatroni777-0:/scripts$ envdir /config /scripts/postgres_backup.sh /home/postgres/pgdata/pgroot/data
2023-08-17 12:14:53.934 - /scripts/postgres_backup.sh - I was called as: /scripts/postgres_backup.sh /home/postgres/pgdata/pgroot/data
2023-08-17 12:14:54.000 - /scripts/postgres_backup.sh - producing a new backup
INFO: 2023/08/17 12:14:54.045921 Calling pg_start_backup()
INFO: 2023/08/17 12:14:54.361692 Initializing the PG alive checker (interval=5m0s)...
INFO: 2023/08/17 12:14:54.361729 Starting a new tar bundle
INFO: 2023/08/17 12:14:54.361756 Walking ...
INFO: 2023/08/17 12:14:54.361965 Starting part 1 ...
INFO: 2023/08/17 12:14:54.503048 Packing ...
INFO: 2023/08/17 12:14:54.503773 Finished writing part 1.
INFO: 2023/08/17 12:14:54.503804 Starting part 2 ...
INFO: 2023/08/17 12:14:54.503815 /global/pg_control
INFO: 2023/08/17 12:14:54.503957 Finished writing part 2.
INFO: 2023/08/17 12:14:54.503961 Calling pg_stop_backup()
INFO: 2023/08/17 12:14:54.624791 Starting part 3 ...
INFO: 2023/08/17 12:14:54.625013 backup_label
INFO: 2023/08/17 12:14:54.625022 tablespace_map
INFO: 2023/08/17 12:14:54.625069 Finished writing part 3.
INFO: 2023/08/17 12:14:54.629729 Wrote backup with name base_00000002000000000000000C
--------------------------------------

#смотрим какие бекапы у нас есть в наличии
--------------------------------------
envdir /config /usr/local/bin/wal-g backup-list
name                          modified             wal_segment_backup_start
base_00000002000000000000000A 2023-08-17T12:13:49Z 00000002000000000000000A
base_00000002000000000000000C 2023-08-17T12:14:54Z 00000002000000000000000C
base_000000020000000000000012 2023-08-17T13:30:34Z 000000020000000000000012
--------------------------------------


#критическая ситуация удаляю данные со всех дисков patroni
#удаляю данные
rm -rf /var/lib/pgsql/15/data
rm -rf /var/lib/pgsql/15/data



#получить последний бекап из wal-g
envdir /config /usr/local/bin/wal-g backup-fetch /home/postgres/pgdata/pgroot/data LATEST
envdir /config /usr/local/bin/wal-g backup-fetch /home/postgres/pgdata/pgroot/data LATEST

-----------------------------------------
2023-08-17 13:33:07,792 INFO: Lock owner: zalandopatroni777-1; I am zalandopatroni777-0
2023-08-17 13:33:07,792 INFO: bootstrap from leader 'zalandopatroni777-1' in progress
pg_basebackup: error: could not initiate base backup: ERROR:  the standby was promoted during online backup
HINT:  This means that the backup being taken is corrupt and should not be used. Try taking another online backup.
pg_basebackup: removing data directory "/home/postgres/pgdata/pgroot/data"
2023-08-17 13:33:09,001 INFO: Lock owner: zalandopatroni777-1; I am zalandopatroni777-0
2023-08-17 13:33:09,001 INFO: bootstrap from leader 'zalandopatroni777-1' in progress
pg_basebackup: error: connection to server at "10.244.2.37", port 5432 failed: FATAL:  could not open file "global/pg_filenode.map": No such file or directory
connection to server at "10.244.2.37", port 5432 failed: FATAL:  could not open file "global/pg_filenode.map": No such file or directory
2023-08-17 13:33:19,476 INFO: Lock owner: None; I am zalandopatroni777-0
2023-08-17 13:33:19,476 INFO: not healthy enough for leader race
2023-08-17 13:33:19,476 INFO: bootstrap from leader 'zalandopatroni777-1' in progress
2023-08-17 13:33:22.403 UTC [27] LOG {ticks: 0, maint: 0, retry: 0}
2023-08-17 13:33:29,976 INFO: Lock owner: None; I am zalandopatroni777-0
2023-08-17 13:33:29,976 INFO: not healthy enough for leader race
2023-08-17 13:33:29,976 INFO: bootstrap from leader 'zalandopatroni777-1' in progress
pg_basebackup: error: connection to server at "10.244.2.37", port 5432 failed: Connection refused
        Is the server running on that host and accepting TCP/IP connections?
2023-08-17 13:33:38,139 ERROR: Error creating replica using method basebackup_fast_xlog: /scripts/basebackup.sh exited with code=1
2023-08-17 13:33:38,139 ERROR: failed to bootstrap from leader 'zalandopatroni777-1'
2023-08-17 13:33:38,139 INFO: Removing data directory: /home/postgres/pgdata/pgroot/data
2023-08-17 13:33:39,976 INFO: Lock owner: None; I am zalandopatroni777-0
2023-08-17 13:33:40,092 INFO: waiting for leader to bootstrap
2023-08-17 13:33:49,976 INFO: Lock owner: None; I am zalandopatroni777-0
2023-08-17 13:33:49,976 INFO: waiting for leader to bootstrap
2023-08-17 13:33:52.406 UTC [27] ERROR connection error: PQconnectStart
2023-08-17 13:33:52.406 UTC [27] ERROR libpq: connection to server on socket "/var/run/postgresql/.s.PGSQL.5432" failed: No such file or directory
                Is the server running locally and accepting connections on that socket?
2023-08-17 13:33:52.406 UTC [27] ERROR connection error: PQconnectStart
2023-08-17 13:33:52.406 UTC [27] ERROR libpq: connection to server on socket "/var/run/postgresql/.s.PGSQL.5432" failed: No such file or directory
                Is the server running locally and accepting connections on that socket?
2023-08-17 13:33:52.406 UTC [27] WARNING [yoyku]: default timeout
2023-08-17 13:33:52.406 UTC [27] ERROR connection error: PQconnectStart
2023-08-17 13:33:52.406 UTC [27] ERROR libpq: connection to server on socket "/var/run/postgresql/.s.PGSQL.5432" failed: No such file or directory
------------------------------------------


### список бекапов
envdir /config /usr/local/bin/wal-g backup-list
envdir /config /usr/local/bin/wal-g backup-list

name                          modified             wal_segment_backup_start
base_00000002000000000000000A 2023-08-17T12:13:49Z 00000002000000000000000A
base_00000002000000000000000C 2023-08-17T12:14:54Z 00000002000000000000000C


### восстанавливаюсь на определенный бекап
/usr/local/bin/wal-g/wal-g-pg backup-fetch /var/lib/pgsql/15/data backup_name

envdir /config /usr/local/bin/wal-g backup-fetch /home/postgres/pgdata/pgroot/data base_000000020000000000000012
envdir /config /usr/local/bin/wal-g backup-fetch /home/postgres/pgdata/pgroot/data base_000000020000000000000012


#видим очень интересную картину
[root@masterkub all_db_postgre_backup_2023_08_17_13:18]# kubectl -n spilo exec -it pod/zalandopatroni777-0  -- patronictl list
+ Cluster: zalandopatroni777 -------+---------+----------+----+-----------+
| Member              | Host        | Role    | State    | TL | Lag in MB |
+---------------------+-------------+---------+----------+----+-----------+
| zalandopatroni777-0 | 10.244.1.38 | Replica | starting |    |   unknown |
| zalandopatroni777-1 | 10.244.2.37 | Replica | stopped  |    |   unknown |
| zalandopatroni777-2 | 10.244.3.42 | Replica | stopped  |    |   unknown |
+---------------------+-------------+---------+----------+----+-----------+


var/run/postgresql:5432 - rejecting connections
2023-08-17 13:41:39,982 INFO: Lock owner: None; I am zalandopatroni777-0
2023-08-17 13:41:39,983 INFO: Still starting up as a standby.
2023-08-17 13:41:39,983 INFO: establishing a new patroni connection to the postgres cluster
2023-08-17 13:41:40,296 INFO: establishing a new patroni connection to the postgres cluster
2023-08-17 13:41:40,297 WARNING: Retry got exception: 'connection problems'
2023-08-17 13:41:40,297 WARNING: Failed to determine PostgreSQL state from the connection, falling back to cached role
2023-08-17 13:41:40,298 INFO: following a different leader because i am not the healthiest node
/var/run/postgresql:5432 - rejecting connections
2023-08-17 13:41:49,982 INFO: Lock owner: None; I am zalandopatroni777-0
2023-08-17 13:41:49,982 INFO: Still starting up as a standby.
2023-08-17 13:41:49,983 INFO: establishing a new patroni connection to the postgres cluster
2023-08-17 13:41:50,566 INFO: establishing a new patroni connection to the postgres cluster
2023-08-17 13:41:50,568 WARNING: Retry got exception: 'connection problems'
2023-08-17 13:41:50,568 WARNING: Failed to determine PostgreSQL state from the connection, falling back to cached role
2023-08-17 13:41:50,568 INFO: following a different leader because i am not the healthiest node
2023-08-17 13:41:52.553 UTC [27] ERROR connection error: PQconnectPoll
2023-08-17 13:41:52.553 UTC [27] ERROR libpq: connection to server on socket "/var/run/postgresql/.s.PGSQL.5432" failed: FATAL:  the database system is starting up
2023-08-17 13:41:52.553 UTC [27] WARNING [yoyku]: default timeout
2023-08-17 13:41:52.553 UTC [27] ERROR connection error: PQconnectPoll
2023-08-17 13:41:52.553 UTC [27] ERROR libpq: connection to server on socket "/var/run/postgresql/.s.PGSQL.5432" failed: FATAL:  the database system is starting up
2023-08-17 13:41:52.554 UTC [27] ERROR connection error: PQconnectPoll
2023-08-17 13:41:52.554 UTC [27] ERROR libpq: connection to server on socket "/var/run/postgresql/.s.PGSQL.5432" failed: FATAL:  the database system is starting up
2023-08-17 13:41:52.554 UTC [27] WARNING [postgres]: default timeout
2023-08-17 13:41:52.587 UTC [27] LOG {ticks: 0, maint: 0, retry: 0}


#пытаюсь рестартовать кластер не получается
kubectl -n spilo exec -it pod/zalandopatroni777-0  -- patronictl restart zalandopatroni777
kubectl -n spilo exec -it pod/zalandopatroni777-0  -- patronictl restart zalandopatroni777

/var/run/postgresql:5432 - rejecting connections
2023-08-17 13:45:22.628 UTC [27] LOG {ticks: 0, maint: 0, retry: 0}
/var/run/postgresql:5432 - rejecting connections
/var/run/postgresql:5432 - rejecting connections
/var/run/postgresql:5432 - rejecting connections
/var/run/postgresql:5432 - rejecting connections
/var/run/postgresql:5432 - rejecting connections
/var/run/postgresql:5432 - rejecting connections
/var/run/postgresql:5432 - rejecting connections
2023-08-17 13:45:29,976 INFO: Lock owner: None; I am zalandopatroni777-0
2023-08-17 13:45:29,976 INFO: not healthy enough for leader race
 

#пытаюсь сменить лидера
Current cluster topology
+ Cluster: zalandopatroni777 -------+---------+----------+----+-----------+
| Member              | Host        | Role    | State    | TL | Lag in MB |
+---------------------+-------------+---------+----------+----+-----------+
| zalandopatroni777-0 | 10.244.1.38 | Replica | starting |    |   unknown |
| zalandopatroni777-1 | 10.244.2.37 | Replica | stopped  |    |   unknown |
| zalandopatroni777-2 | 10.244.3.42 | Replica | stopped  |    |   unknown |
+---------------------+-------------+---------+----------+----+-----------+
Error: This cluster has no leader
command terminated with exit code 1



#удаляем поды 
kubectl delete pods zalandopatroni777-0 zalandopatroni777-1 zalandopatroni777-2 --namespace spilo
kubectl delete pods zalandopatroni777-0 zalandopatroni777-1 zalandopatroni777-2 --namespace spilo


kubectl -n spilo exec -it pod/zalandopatroni777-1 -- patronictl remove zalandopatroni777
kubectl -n spilo exec -it pod/zalandopatroni777-1 -- patronictl remove zalandopatroni777
kubectl -n spilo exec -it pod/zalandopatroni777-1 -- patronictl remove zalandopatroni777




###ЕСЛИ НАДО СДЕЛАТЬ ВОССТАНОВЛЕНИЕ НА ТОЧКУ ВРЕМЕНИ ПРИ УСЛОВИИ ЧТО КЛАСТЕР НЕ РАБОТОСПОСОБЕН
Предупреждение: ваш каталог /data должен быть доступен для записи пользователю postgres.

Шаги к PITR:

1)
Остановите Patroni на всех узлах реплик и, наконец, на мастере
sudo systemctl stop Patroni

2)
Обновить файл конфигурации /etc/patroni.yml
recovery_target_time: '2020-06-08 08:52:00'

3)
Удалить кластер из etcd
patronictl -c /etc/patroni.yml remove postgres

4)
Резервное копирование и удаление каталога данных на мастере /data/patroni

5)
Запустите Patroni на мастере — он автоматически вызовет скрипт clone_with_walg.sh
sudo systemctl start patchi



#пример который работает с восстановлением
----------------------------------------------------------------------------------------------------------
scope: postgres
namespace: /db/
name: postgresql1

restapi:
    listen: 10.0.1.4:8008
    connect_address: 10.0.1.4:8008

etcd:
    host: 10.0.1.7:2379

bootstrap:
    dcs:
        ttl: 30
        loop_wait: 10
        retry_timeout: 10
        maximum_lag_on_failover: 1048576
        postgresql:
            use_pg_rewind: true

    method: clone_with_walg
    clone_with_walg:
        command: /home/postgres/clone_with_walg.sh
        recovery_conf:
            restore_command: envdir /etc/wal-g.d/env wal-g wal-fetch "%f" "%p"
            recovery_target_timeline: latest
            recovery_target_action: promote
            recovery_target_time: ''

    initdb:
    - encoding: UTF8
    - data-checksums

    pg_hba:
    - host replication replicator 127.0.0.1/32 md5
    - host replication replicator 10.0.1.4/0 md5
    - host replication replicator 10.0.1.8/0 md5
    - host replication replicator 10.0.1.6/0 md5
    - host all all 0.0.0.0/0 md5

postgresql:
    listen: 10.0.1.4:5432
    connect_address: 10.0.1.4:5432
    data_dir: /data/patroni
    pgpass: /tmp/pgpass
    authentication:
        replication:
            username: replicator
            password: rep-pass
        superuser:
            username: postgres
            password: secretpassword
    parameters:
        unix_socket_directories: '/var/run/postgresql'
        shared_preload_libraries: 'pg_stat_statements'
        archive_mode: 'on'
        archive_timeout: 300s
        archive_command: 'envdir /etc/wal-g.d/env wal-g wal-push %p'
    recovery_conf:
        restore_command: 'envdir /etc/wal-g.d/env wal-g wal-fetch "%f" "%p"'

tags:
    nofailover: false
    noloadbalance: false
    clonefrom: false
    nosync: false
----------------------------------------------------------------------------------------------------------

----------------------------------------------------------------------------------------------------------
И простой скрипт clone_with_walg.sh:

#!/bin/bash

mkdir -p /data/patroni
envdir /etc/wal-g.d/env wal-g backup-fetch /data/patroni LATEST
----------------------------------------------------------------------------------------------------------
</pre>






<pre>
---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------дали нагрузку через custom pgbench -------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
</pre>

<pre>
pgbench -c30 -j 3 -T 600 -f /tmp/my.sql -U postgres 


\set aid random(1, 100000 * :scale)
\set tid random(1, 10 * :scale)

BEGIN;
SELECT abalance FROM pgbench_accounts WHERE aid = :aid;
UPDATE pgbench_tellers SET tbalance = tbalance + :delta WHERE tid = :tid;
END;


latency average = 303.688 ms
latency stddev = 330.130 ms
average connection time = 6.301 ms
tps = 1011.745819 (including reconnection times)



описание ключей pgbench:

c clients
--client=clients Количество смоделированных клиентов, то есть количество одновременных сеансов базы данных. По умолчанию 1.

-C (каждый пользователь подключается под своей сессией)

-P sec
--progress=sec
Показывать отчет о проделанной работе каждую sec секунду. Отчет включает время с начала выполнения, 
TPS с момента последнего отчета, а также среднюю задержку транзакции и стандартное отклонение с момента последнего отчета. 
При регулировании ( -R) задержка вычисляется относительно запланированного времени начала транзакции, 
а не фактического времени начала транзакции, поэтому она также включает среднее время задержки расписания.

-T seconds
--time=seconds
Запустите тест на это количество секунд, а не на фиксированное количество транзакций на клиента. -t и -T являются взаимоисключающими.

-j threads
--jobs=threads
Количество рабочих потоков в pgbench . Использование более одного потока может быть полезно на многопроцессорных машинах. Клиенты распределяются максимально равномерно по доступным потокам. По умолчанию — 1.
</pre>






<pre>
---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------вспомогательные команды-------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
</pre>



для внутрянки:
patronictl -c postgres.yml list
patronictl -c postgres.yml list
patronictl -c postgres.yml list


<pre>
----------------------------------------------------------
kubectl get pods --all-namespaces -o wide
kubectl get pods --all-namespaces -o wide
kubectl get pods --all-namespaces -o wide

### Это предоставит информацию обо всех deployments во всех namespaces
kubectl get deployments --all-namespaces
kubectl get deployments --all-namespaces
kubectl get deployments --all-namespaces

### Это предоставит информацию обо всех deployments в namespaces cert-manager
kubectl get deployments -n cert-manager
kubectl get deployments -n cert-manager
kubectl get deployments -n cert-manager


### Это предоставит информацию обо всех модулях, развертываниях, службах и заданиях в пространстве имен.
kubectl get pods,services,deployments,jobs
kubectl get pods,services,deployments,jobs
kubectl get pods,services,deployments,jobs

### Это предоставит информацию о сервисахесли
kubectl get services
kubectl get services
kubectl get services

### Это предоставит информацию о пространствах имен
kubectl get namespace
kubectl get namespace
kubectl get namespace

### Чтобы перечислить все развертывания:
kubectl get deployments --all-namespaces
kubectl get deployments --all-namespaces
kubectl get deployments --all-namespaces

kubectl get all
kubectl get all
kubectl get all

### показать все поды даже системные
kubectl get all -A
kubectl get all -A
kubectl get all -A

kubectl get all --all-namespaces
kubectl get all --all-namespaces
kubectl get all --all-namespaces

kubectl get pods
kubectl get pods
kubectl get pods

kubectl get nodes
kubectl get nodes
kubectl get nodes

### А вот для проверки PV и PVC необходимо выполнить следующую команду :
kubectl apply -f /tmp/citus/postgres-storage.yaml
kubectl get pvc -o wide
kubectl get pv -o wide

kubectl get pv
kubectl describe pv
kubectl get pvc
kubectl describe pvc
kubectl get pv
kubectl get all -o wide

### посмотрим дефолтный тип стораджа
kubectl get storageclasses

kubectl get pvc -o wide
kubectl get pv -o wide

### удалить все
kubectl delete all,ing,secrets,pvc,pv --all
kubectl delete all,ing,secrets,pvc,pv --all

### прозвонить pod
kubectl describe pod -n kube-system etcd-masterkub.ru-central1.internal
kubectl describe pod -n kube-system etcd-masterkub.ru-central1.internal
kubectl describe pod -n kube-system etcd-masterkub.ru-central1.internal

### логи pod
kubectl logs --namespace kube-system etcd-masterkub.ru-central1.internal
kubectl logs --namespace kube-system etcd-masterkub.ru-central1.internal
kubectl logs --namespace kube-system etcd-masterkub.ru-central1.internal
kubectl logs --namespace kube-system kube-flannel-ds-amd64-5fx2p
kubectl logs --namespace kube-system kube-flannel-ds-amd64-5fx2p
kubectl logs --namespace kube-system kube-flannel-ds-amd64-5fx2p

kubectl logs pod_name(необязательно с -n namespace_name)
kubectl logs pod_name(необязательно с -n namespace_name)
kubectl logs pod_name(необязательно с -n namespace_name)

###### смотреть логи в режиме реального времени pod pod/zalandopatroni01-0 namespace spilo используя префикс -f
kubectl logs --namespace spilo pod/zalandopatroni01-0 -f
kubectl logs --namespace spilo pod/zalandopatroni01-0 -f
kubectl logs --namespace spilo pod/zalandopatroni01-0 -f

https://sematext.com/blog/tail-kubernetes-logs/
https://sematext.com/blog/tail-kubernetes-logs/
https://sematext.com/blog/tail-kubernetes-logs/


### Запустите команду ниже, чтобы получить события. Это покажет проблему (и все другие события), почему модуль не запланирован.
kubectl get events
kubectl get events
kubectl get events

---
### смотрим интерфейсы
ip -4 addr show
ip -4 addr show
ip -4 addr show

### Удалите интерфейс cni0и файлы flannel.1.
sudo ip link delete cni0;
sudo ip link delete flannel.1;
---

tail -f /var/log/messages
tail -f /var/log/messages
tail -f /var/log/messages


### требуется удалить ненужные поды
----------------------------------------------------------------------------------------------------------------------------
https://stackoverflow.com/questions/40686151/kubernetes-pod-gets-recreated-when-deleted
https://stackoverflow.com/questions/40686151/kubernetes-pod-gets-recreated-when-deleted
https://stackoverflow.com/questions/40686151/kubernetes-pod-gets-recreated-when-deleted
------------
kubectl delete -f k8s
kubectl delete -f k8s
kubectl delete -f k8s
[root@masterkub citus-test]# kubectl delete -f k8s
statefulset.apps "citus" deleted
configmap "pg-hba" deleted
statefulset.apps "citus-coordinator-replica" deleted
statefulset.apps "citus-coordinator" deleted
statefulset.apps "citus-worker" deleted
------------
------------
kubectl delete all --all
-- не забываем про:
kubectl get pvc -o wide
kubectl get pv -o wide
kubectl get cm -o wide
kubectl get secrets -o wide
kubectl delete pvc --all
kubectl delete cm --all
kubectl delete secrets --all

kubectl delete all --all && kubectl delete ing --all && kubectl delete secrets --all && kubectl delete pvc --all && kubectl delete pv --all
kubectl delete all --all && kubectl delete ing --all && kubectl delete secrets --all && kubectl delete pvc --all && kubectl delete pv --all
kubectl delete all --all && kubectl delete ing --all && kubectl delete secrets --all && kubectl delete pvc --all && kubectl delete pv --all

-- ну или так)
kubectl delete all,ing,secrets,pvc,pv --all
kubectl delete all,ing,secrets,pvc,pv --all
kubectl delete all,ing,secrets,pvc,pv --all

kubectl delete pvc --all

-- удалим все
kubectl delete all,ing,secrets,pvc,pv --all
kubectl delete deployment --all
------------
------------
Это предоставит информацию обо всех модулях, развертываниях, службах и заданиях в пространстве имен.

kubectl get pods,services,deployments,jobs
kubectl get pods,services,deployments,jobs
kubectl get pods,services,deployments,jobs

------------
------------
удалить конкретный под с именем zalandopatroni01-0 который расположен в namespace spilo
kubectl delete pod zalandopatroni01-0 --namespace spilo
kubectl delete pod zalandopatroni01-0 --namespace spilo
kubectl delete pod zalandopatroni01-0 --namespace spilo
------------
------------
модули могут быть созданы развертываниями или заданиями

kubectl delete job [job_name]
kubectl delete deployment [deployment_name]
Если вы удалите развертывание или задание, перезапуск модулей может быть остановлен.
------------
----------------------------------------------------------
</pre>



<pre>
---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------на почитать-------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
</pre>


<pre>
Настройка кластера Kubernetes Centos 7
https://www.youtube.com/watch?v=5JBLITD3u6I
https://www.youtube.com/watch?v=5JBLITD3u6I
https://www.youtube.com/watch?v=5JBLITD3u6I

https://www.youtube.com/watch?v=XfWE7NqgbLE&list=PLmxqUDFl0XM6pr2y6tK51cHOFajhFsWG8&index=4
https://www.youtube.com/watch?v=XfWE7NqgbLE&list=PLmxqUDFl0XM6pr2y6tK51cHOFajhFsWG8&index=4
https://www.youtube.com/watch?v=XfWE7NqgbLE&list=PLmxqUDFl0XM6pr2y6tK51cHOFajhFsWG8&index=4 

Kubernetes 1.26 — The electrifying release
https://blog.kubesimplify.com/kubernetes-126
https://blog.kubesimplify.com/kubernetes-126
https://blog.kubesimplify.com/kubernetes-126

What is Citus?
https://github.com/citusdata/citus
https://github.com/citusdata/citus
https://github.com/citusdata/citus

manual Citusdata to GKE
https://github.com/citusdata/citus/issues/425
https://github.com/citusdata/citus/issues/425
https://github.com/citusdata/citus/issues/425

Шпаргалка по kubectl
https://kubernetes.io/ru/docs/reference/kubectl/cheatsheet/
https://kubernetes.io/ru/docs/reference/kubectl/cheatsheet/
https://kubernetes.io/ru/docs/reference/kubectl/cheatsheet/

PVC. Как создать и привязать к поду
https://support.edgecenter.ru/knowledge_base/item/289604
https://support.edgecenter.ru/knowledge_base/item/289604
https://support.edgecenter.ru/knowledge_base/item/289604

добавить метку к узлу
https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes-using-node-affinity/
https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes-using-node-affinity/
https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes-using-node-affinity/

релизы кубера
https://kubernetes.io/releases/
https://kubernetes.io/releases/
https://kubernetes.io/releases/

релизы citus
https://github.com/citusdata/docker/blob/master/CHANGELOG.md
https://github.com/citusdata/docker/blob/master/CHANGELOG.md
https://github.com/citusdata/docker/blob/master/CHANGELOG.md


Patroni в GKE // Демо-занятие курса «PostgreSQL Cloud Solutions»
https://www.youtube.com/watch?v=8eTnvIFYERQ
https://www.youtube.com/watch?v=8eTnvIFYERQ
https://www.youtube.com/watch?v=8eTnvIFYERQ
</pre>
