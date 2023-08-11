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

#### 5.6)
#### извлекаем образы для версии Kubernetes 1.26. Pull the images

<pre>
------------------------------
sudo kubeadm config images pull --image-repository=registry.k8s.io --cri-socket unix:///run/containerd/containerd.sock --kubernetes-version v1.26.1
sudo kubeadm config images pull --image-repository=registry.k8s.io --cri-socket unix:///run/containerd/containerd.sock --kubernetes-version v1.26.1
sudo kubeadm config images pull --image-repository=registry.k8s.io --cri-socket unix:///run/containerd/containerd.sock --kubernetes-version v1.26.1
------------------------------
</pre>

#### 5.7)

#### запускаю команду инициализации kubeadm на узле управления.


#### Здесь CIDR сети pod зависит от CNI, который вы будете устанавливать позже, поэтому в этом случае я использую фланель 
#### и --control-plane-endpoint буду общедоступным IP-адресом для экземпляра (это также может быть частный IP-адрес, 
#### но если вы хотите получить доступ это из-за пределов узла с помощью Kubeconfig, тогда вам нужно указать общедоступный IP-адрес).

<pre>
------------------------------
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=10.129.0.24 --upload-certs --kubernetes-version=v1.26.1 --control-plane-endpoint=masterkub.ru-central1.internal --cri-socket unix:///run/containerd/containerd.sock
------------------------------
</pre>

#### 5.8)

#### проверяю

<pre>
------------------------------
kubectl version
kubectl version
kubectl version

WARNING: This version information is deprecated and will be replaced with the output from kubectl version --short.  Use --output=yaml|json to get the full version.
Client Version: version.Info{Major:"1", Minor:"26", GitVersion:"v1.26.1", GitCommit:"8f94681cd294aa8cfd3407b8191f6c70214973a4", GitTreeState:"clean", BuildDate:"2023-01-18T15:58:16Z", GoVersion:"go1.19.5", Compiler:"gc", Platform:"linux/amd64"}
Kustomize Version: v4.5.7
Server Version: version.Info{Major:"1", Minor:"26", GitVersion:"v1.26.1", GitCommit:"8f94681cd294aa8cfd3407b8191f6c70214973a4", GitTreeState:"clean", BuildDate:"2023-01-18T15:51:25Z", GoVersion:"go1.19.5", Compiler:"gc", Platform:"linux/amd64"}
------------------------------

------------------------------
hostname -i
hostname -i
hostname -i
------------------------------
</pre>

#### 5.9)

#### выполняю init Kubernetes cluster

<pre>
------------------------------
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=10.129.0.24 --upload-certs --kubernetes-version=v1.26.1 --control-plane-endpoint=masterkub.ru-central1.internal --cri-socket unix:///run/containerd/containerd.sock
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=10.129.0.24 --upload-certs --kubernetes-version=v1.26.1 --control-plane-endpoint=masterkub.ru-central1.internal --cri-socket unix:///run/containerd/containerd.sock
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=10.129.0.24 --upload-certs --kubernetes-version=v1.26.1 --control-plane-endpoint=masterkub.ru-central1.internal --cri-socket unix:///run/containerd/containerd.sock
[config/images] Pulled registry.k8s.io/kube-apiserver:v1.26.1
[config/images] Pulled registry.k8s.io/kube-controller-manager:v1.26.1
[config/images] Pulled registry.k8s.io/kube-scheduler:v1.26.1
[config/images] Pulled registry.k8s.io/kube-proxy:v1.26.1
[config/images] Pulled registry.k8s.io/pause:3.9
[config/images] Pulled registry.k8s.io/etcd:3.5.6-0
[config/images] Pulled registry.k8s.io/coredns/coredns:v1.9.3
[root@masterkub modules-load.d]# sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=10.129.0.24 --upload-certs --kubernetes-version=v1.26.1 --control-plane-endpoint=masterkub.ru-central1.internal --cri-socket unix:///run/containerd/containerd.sock
[init] Using Kubernetes version: v1.26.1
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local masterkub.ru-central1.internal] and IPs [10.96.0.1 10.129.0.24]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [localhost masterkub.ru-central1.internal] and IPs [10.129.0.24 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [localhost masterkub.ru-central1.internal] and IPs [10.129.0.24 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 7.502577 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Storing the certificates in Secret "kubeadm-certs" in the "kube-system" Namespace
[upload-certs] Using certificate key:
25e511511f5ae6e5c0428dd276919391b4cb5692d32b9593627f9152715d5d97
[mark-control-plane] Marking the node masterkub.ru-central1.internal as control-plane by adding the labels: [node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node masterkub.ru-central1.internal as control-plane by adding the taints [node-role.kubernetes.io/control-plane:NoSchedule]
[bootstrap-token] Using token: eo5469.fboaarc5jjvl4dh1
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

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

You can now join any number of the control-plane node running the following command on each as root:

  kubeadm join masterkub.ru-central1.internal:6443 --token eo5469.fboaarc5jjvl4dh1 \
        --discovery-token-ca-cert-hash sha256:ab7b5ce50b4825850aa86009b2f1d9a7c67db6ff872b08575f691410f8f4b220 \
        --control-plane --certificate-key 25e511511f5ae6e5c0428dd276919391b4cb5692d32b9593627f9152715d5d97

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join masterkub.ru-central1.internal:6443 --token eo5469.fboaarc5jjvl4dh1 \
        --discovery-token-ca-cert-hash sha256:ab7b5ce50b4825850aa86009b2f1d9a7c67db6ff872b08575f691410f8f4b220
------------------------------
</pre>


#### 5.10)

#### основные конфиги (для распространения управления на других узлах)
#### Экспортируйте KUBECONFIG и установите CNI Flannel.

<pre>
------------------------------
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
export KUBECONFIG=/etc/kubernetes/admin.conf

настройка сети
kubectl apply -f https://github.com/coreos/flannel/raw/master/Documentation/kube-flannel.yml
kubectl apply -f https://github.com/coreos/flannel/raw/master/Documentation/kube-flannel.yml
kubectl apply -f https://github.com/coreos/flannel/raw/master/Documentation/kube-flannel.yml
------------------------------
</pre>

#### 5.11)

#### ловим ошибку CrashLoopBackOff

<pre>
----------------------------------------------------------------------------------------------------------------------------
https://stackoverflow.com/questions/52098214/kube-flannel-in-crashloopbackoff-status
https://stackoverflow.com/questions/52098214/kube-flannel-in-crashloopbackoff-status
https://stackoverflow.com/questions/52098214/kube-flannel-in-crashloopbackoff-status

ошибка в моем случае:
kube-system   kube-flannel-ds-amd64-42rl7            0/1       CrashLoopBackOff
как оказалась проблема была в этом --pod-network-cidr=10.244.0.0/16
Для flannelкорректной работы необходимо перейти --pod-network-cidr=10.244.0.0/16на kubeadm init.
Это верно только для главного узла. Другие рабочие узлы не должны запускать эту команду. 
Посмотрите на другой ответ, предоставленный @pande ниже; это то, что решило эту проблему для меня.

как можно разрулить дополнительно:
------------------------
Причина в том, что
фланелевый бег с CIDR= 10.244.0.0/16​​НЕТ 10.244.0.0/24!!!
Конфликты CNI из-за того, что узел установил несколько подключаемых модулей CNI в /etc/cni/net.d/.
Интерфейс 2 flannel.1и cni0не соответствовали друг другу. Например:
flannel.1=10.244.0.0и cni0=10.244.1.1потерпит неудачу. Это должно быть
flannel.1=10.244.0.0иcni0=10.244.0.1

Чтобы исправить это, выполните следующие действия:

Шаг 0: Сбросьте все узлы в вашем кластере. Запустите все узлы с
kubeadm reset --force;

Шаг 1: вниз по интерфейсу cni0и flannel.1.
sudo ifconfig cni0 down;
sudo ifconfig flannel.1 down;

Шаг 2: Удалите интерфейс cni0и файлы flannel.1.
sudo ip link delete cni0;
sudo ip link delete flannel.1;

Шаг 3: Удалите все элементы внутри /etc/cni/net.d/.
sudo rm -rf /etc/cni/net.d/;

Шаг 4. Повторно загрузите кластер Kubernetes.
kubeadm init --control-plane-endpoint="..." --pod-network-cidr=10.244.0.0/16;

Шаг 5. Повторно разверните CNI.
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml;

Шаг 6: Перезапустите ваши CNI, здесь я использовал Container Daemon (Containerd).
systemctl restart containerd;

Это обеспечит правильную работу вашего Core-DNS.
----------------------------------------------------------------------------------------------------------------------------
</pre>

#### 5.12)

#### проверяем статус и что все работает. a ..

<pre>
----------------------------------------------------------------------------------------------------------------------------
root@masterkub vorori]# kubectl get nodes
NAME                             STATUS   ROLES           AGE     VERSION
masterkub.ru-central1.internal   Ready    control-plane   4m27s   v1.26.1
----------------------------------------------------------------------------------------------------------------------------
</pre>

#### 5.13)

#### смотрим что все работает. b ..

<pre>
----------------------------------------------------------------------------------------------------------------------------
[root@masterkub ~]# kubectl get all -A
NAMESPACE      NAME                                                         READY   STATUS    RESTARTS   AGE
kube-flannel   pod/kube-flannel-ds-rdp9t                                    1/1     Running   0          59s
kube-system    pod/coredns-787d4945fb-f98p8                                 1/1     Running   0          3m27s
kube-system    pod/coredns-787d4945fb-s78vs                                 1/1     Running   0          3m27s
kube-system    pod/etcd-masterkub.ru-central1.internal                      1/1     Running   1          3m41s
kube-system    pod/kube-apiserver-masterkub.ru-central1.internal            1/1     Running   1          3m39s
kube-system    pod/kube-controller-manager-masterkub.ru-central1.internal   1/1     Running   0          3m39s
kube-system    pod/kube-proxy-9khtr                                         1/1     Running   0          3m27s
kube-system    pod/kube-scheduler-masterkub.ru-central1.internal            1/1     Running   1          3m39s

NAMESPACE     NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
default       service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP                  3m41s
kube-system   service/kube-dns     ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   3m39s

NAMESPACE      NAME                             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-flannel   daemonset.apps/kube-flannel-ds   1         1         1       1            1           <none>                   59s
kube-system    daemonset.apps/kube-proxy        1         1         1       1            1           kubernetes.io/os=linux   3m39s

NAMESPACE     NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
kube-system   deployment.apps/coredns   2/2     2            2           3m39s

NAMESPACE     NAME                                 DESIRED   CURRENT   READY   AGE
kube-system   replicaset.apps/coredns-787d4945fb   2         2         2       3m28s
----------------------------------------------------------------------------------------------------------------------------
</pre>

#### 5.14)

#### на нодах пытаемся присоединиться к мастеру

<pre>
# еа эту ошибку не обрашаем внимания
----------------------------------------------------------------------------------------------------------------------------
rm /etc/containerd/config.toml
rm /etc/containerd/config.toml
rm /etc/containerd/config.toml

systemctl restart containerd && sudo systemctl enable --now containerd && systemctl status containerd
systemctl restart containerd && sudo systemctl enable --now containerd && systemctl status containerd
systemctl restart containerd && sudo systemctl enable --now containerd && systemctl status containerd


kubeadm join masterkub.ru-central1.internal:6443 --token eo5469.fboaarc5jjvl4dh1 --discovery-token-ca-cert-hash sha256:ab7b5ce50b4825850aa86009b2f1d9a7c67db6ff872b08575f691410f8f4b220
kubeadm join masterkub.ru-central1.internal:6443 --token eo5469.fboaarc5jjvl4dh1 --discovery-token-ca-cert-hash sha256:ab7b5ce50b4825850aa86009b2f1d9a7c67db6ff872b08575f691410f8f4b220


---------------------------------
[root@kub1 vorori]# kubeadm join masterkub.ru-central1.internal:6443 --token 6ar5s6.jvgjuszqhzdlmj33 \
>         --discovery-token-ca-cert-hash sha256:bbc69ceafaa3261fc3d7ce088f9f0837318d2e1dbc5f426b7de264f9c5bf64db
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
---------------------------------

---------------------------------
[root@kub2 vorori]# kubeadm join masterkub.ru-central1.internal:6443 --token 6ar5s6.jvgjuszqhzdlmj33 --discovery-token-ca-cert-hash sha256:bbc69ceafaa3261fc3d7ce088f9f0837318d2e1dbc5f426b7de264f9c5bf64db
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
---------------------------------

---------------------------------
[root@kub3 vorori]# kubeadm join masterkub.ru-central1.internal:6443 --token 6ar5s6.jvgjuszqhzdlmj33 --discovery-token-ca-cert-hash sha256:bbc69ceafaa3261fc3d7ce088f9f0837318d2e1dbc5f426b7de264f9c5bf64db
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
---------------------------------
</pre>


#### 5.15)

#### заметка если сделать в ручника (пытаемся присоединиться к мастеру)

<pre>
---------------------------------
можно скопировать файл kubeconfig с узла controlplane node управления (~/.kube/config ) 
на локальный и экспортировать переменную KUBECONFIG или получить прямой доступ к кластеру с узла controlplane node.

mkdir -p $HOME/.kube
mkdir -p $HOME/.kube
mkdir -p $HOME/.kube

cat /etc/kubernetes/admin.conf
cat /etc/kubernetes/admin.conf
cat /etc/kubernetes/admin.conf

apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUMvakNDQWVhZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJek1EZ3hNVEE0TXprek5Gb1hEVE16TURnd09EQTRNemt6TkZvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTDlBCklLTE5jaUJvWDhxS1QzYys2N3prdFZWdEl6d2k2d1o4WUJBZzFNNVI2T3pkQnZaekZsOGtzWSt4ejVHWHR4YmsKWERkekNPSldlSFdTSlR5WXoxT3RvdS9IS2JOdmNETHg1SVo5Mk9YVFhTS0lpcTh2UUFKT1ZpdytTUnNPQ0FqWgppRkVwUUVEY25aN05ZaTJSM1IyWFRUSHpCaDFKb3BWOVcxelYzN25ydksrZFpnMFlUOFd6Mi9OUmdmd2xqU05HCjBIeEhpZk1XY3lTcTJEalRyTmsyTkkwYk9XVmIvK0prNUJGT1dtMUtHSUhOTkl2MXMxRUU3TGhldGRCRXRuQjMKdGUyaCtnS0ROU3kxb1dzekRCNUNzdG9uUWhOVklMS0NLWnc0YWtOSXNBYzhKWFY5eDZqT29mdzlGNDF6dlVVUApuVmdjYy9PeVg4cUROVmdXNXFzQ0F3RUFBYU5aTUZjd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZJUFRlTVBsUWx5VHg2R1k0S1J5Q202Wm83VVpNQlVHQTFVZEVRUU8KTUF5Q0NtdDFZbVZ5Ym1WMFpYTXdEUVlKS29aSWh2Y05BUUVMQlFBRGdnRUJBSEF5bGpKUHhpR0FHNDIwYlhIKwppWlJ2VUEreXY4b0NyaHIzbHJNRHVSV3lBc0tMUnVZY2lseFpXN2ZFQjkxbTdZUS9Lamc2S01vSFhoYlVIZS9aCk93ZVE1NUZzMmNoQmhqQUt6eDltQkZJUzVvWGxNdk9CWllaMWNsVTdyTGI2WGc1UUhKQnpOZ2dXWGhHQ2ZnakwKR1drSk1JWGk1b1E2RFNCclFtQWJkM0wvWHc3eXNjMUUxdXJtaHArb202QUlJTTdxYVdTYkNTd2VwVVVSU05mMApud01la1NyWGJMU3YxZkxVNVlMcUlJdXVKdmFiRFYyYUluMThoN2VialB6ZFNxNm1CT1Z0bmZ6MTg0YVVrbUx5CkxKN0h1ajdobjlGNmRmcTBPdXhNUlBGZkZsRGlML1g2NnpiZU45S3RDUzJ1bGRjTHg2Myt0c2U4RHFBU3hTV0wKZ2hjPQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    server: https://masterkub.ru-central1.internal:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURJVENDQWdtZ0F3SUJBZ0lJZDg2MUN2YWdFUll3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TXpBNE1URXdPRE01TXpSYUZ3MHlOREE0TVRBd09ETTVNelphTURReApGekFWQmdOVkJBb1REbk41YzNSbGJUcHRZWE4wWlhKek1Sa3dGd1lEVlFRREV4QnJkV0psY201bGRHVnpMV0ZrCmJXbHVNSUlCSWpBTkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQXhQQy9nb1NBbERVSStvcjYKOXR0TkpIaHpFZkw4bmk1aXo2S1NBaXc2OGJyR0dFekRjeFNJU0pNMGlqM0wvcUFqbitIZzl3UktleWZJeTcvVgphR3dIaGVMaFdsTjlQdEdYWWd5QTF4Z2RVTFNuOTlhMmY2S2NWb2p0R0Q4ZVdMNm9BWE5UVHFVbmpTTVhxK0k3CmZCUVFiYllPOHFqUmt1UnVLbnMvdkJhei9QRmlRUGJTU1lzbnhNOXJNQkVmbHJVdU9KbXNXMGNxMStZanBNQXcKODUxMnZlUDU0MldxMlBheVhFcDQ0MHlrc0MwRy9DQmkrVXY5SjQ0cVF0SjMwV1QrOXo2ZXFSVVFXYWphY3NhUApOZ0dPQjFVU2JML3pOcWdKUDRlVG4wRDZrNFZTNnhSb0VuNEJRSzBnK0doeEc5enhhVXRZaHpRNHo3eGU3YTZTCkhOT0pJd0lEQVFBQm8xWXdWREFPQmdOVkhROEJBZjhFQkFNQ0JhQXdFd1lEVlIwbEJBd3dDZ1lJS3dZQkJRVUgKQXdJd0RBWURWUjBUQVFIL0JBSXdBREFmQmdOVkhTTUVHREFXZ0JTRDAzakQ1VUpjazhlaG1PQ2tjZ3B1bWFPMQpHVEFOQmdrcWhraUc5dzBCQVFzRkFBT0NBUUVBVVJLRjVZNnNNSm4xbWFiQWlaSlRtMndpRHFkTzhKckVjSFFqClBVY05XYWdJUFVibHViYmJUcFg5VmdqUEk4TDlBb3NJMUFzT2d0Q2dycTZ6NEVtV1NHVnZhbTFOeGxwYWxoSUcKS2lWNkNGbjU2QlRhM0FPVFprRHErem1VWFVaODJOOVNRVDQySkc2cUZnZXhCMVcyS0xVZmhyM2tZVmxJOGFoOQpnT1Rxekx6SjhoU1JtdVAvK1RkRVQ1MDR3NzNhRVFWMHN5cmU1YXpoWFhhb2h4bWE2WklRZVlrTkc2bnlBc0hyCm5jdHc0N0w3TFcrenRFdWtuMTdEbjUxRGJRd2Q4aXFWL3h3L2JDQTk1bEQyU2E5VXFtUktkcG1zaWZXN0JScWQKWWhhVlBEZmhrdG1zR1lPWlFSTTJXL0xTZVF2MGJvcFVoWUFncW5Mc0Z4YU45THB2TWc9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcFFJQkFBS0NBUUVBeFBDL2dvU0FsRFVJK29yNjl0dE5KSGh6RWZMOG5pNWl6NktTQWl3NjhickdHRXpECmN4U0lTSk0waWozTC9xQWpuK0hnOXdSS2V5Zkl5Ny9WYUd3SGhlTGhXbE45UHRHWFlneUExeGdkVUxTbjk5YTIKZjZLY1ZvanRHRDhlV0w2b0FYTlRUcVVualNNWHErSTdmQlFRYmJZTzhxalJrdVJ1S25zL3ZCYXovUEZpUVBiUwpTWXNueE05ck1CRWZsclV1T0ptc1cwY3ExK1lqcE1Bdzg1MTJ2ZVA1NDJXcTJQYXlYRXA0NDB5a3NDMEcvQ0JpCitVdjlKNDRxUXRKMzBXVCs5ejZlcVJVUVdhamFjc2FQTmdHT0IxVVNiTC96TnFnSlA0ZVRuMEQ2azRWUzZ4Um8KRW40QlFLMGcrR2h4Rzl6eGFVdFloelE0ejd4ZTdhNlNITk9KSXdJREFRQUJBb0lCQVFDTkIyWHMvaHZoaGhVVwo3WDJJVjBUbjBBVi9IZ1UrOVRLM1E5RFJFNEZtWjN6Q0cvNStvMzV3a2xHMmlVaFMzN1NESXNycHVUM284WFYrClNySjFJNTlEaGxRZ1RkZExxK1YvUmpyaFRSaDVHZFdLeWt4SUhGZGVOSkdzb2s2RitJbnc5L2Y1UXBXUElVa3IKUWtlY3MvV0x5eXJySkc5bmhJTkxrcFR5aVVOODVzN0duRlNOTDVobFR6VXh0c2RpanpLd2JnOWhjNnRGTWhtbgpEcncvSDJyekJVR1dxcGdXQWhYb0tBOGRPVmFLbFFhYm14NWZrbHVYS1JTYkg3ZXUwRXNwdEJMVnZNckdSeGVxCkhHYkZUTXRLaDVSSEFZQWpZL2JOKzIycjFjTXhVTWpxV0hYdkt1ZGxVd0tmTXpwM0xIaFlNNmR5Z2toYzZkRDgKSGdUcnB3cTVBb0dCQVB5YlpiU2xGaUlNdXRud20vcW5iM2I3YkVnZnYxZmxxNFlIckE3Z2ZVTThTZ3Z0bnNZZwp1R3YxYUl0N0lMcGhPYmQxZG9ydGh6SVJVS0FHbFlOTUlNaDdSajVKTC9EUkVmK2pJZmQ3aVlvWFAycFd1YXNRClRXN0praDJDN3l2a0VpZmFRMHJTRU1VV2FiOFk5RlRQVk9QYTZtRm5MSm1YamVSNXJjMFNIbk90QW9HQkFNZVYKOEMvWkI2T2hqZG83MTFqSDVYS2tKNXJocmJXUDFUMGQ5UjJJR0dKUkxZR2JJNEcwdnV2L2RacXBTSUZYZ0NBdQpJNE9Eclk5enRlRkZjRWs1dTlaTFJoYWF6RlFIdzNzY1ZsbDFOTGRSVlhJK2FjaEl0LzBmVVpiejRBd0pCZ2JwCkU1TUFXWS9YT2srdURqaks3NitxQ09JRG15KzBvSE9xN3RoUFhnb1BBb0dCQU5RREpTaXB5bHJIcm1mTzMwdFEKRG1paGV1OUozaEhLek54UVFpTzJYTXY2cFBjLzk1dTR5TENycDVReHduVkx0dUo0cndiSmQwZ1phajcxWjdWcwpSck9kYTRaSmJQaEVzVU9LeXE1cFBEWHZieVUwSnQ4aGJxd0dlQ0ZXektCYzZyUVNKNXA3bHVHai94c0p1Y0FZCng5bjUyZS9vWlhGLzF2S2xBYTkxZnFOOUFvR0FWeURybzllNDhBUWM2d0pvdGtjOXNWaGNPYzcvaUYxc0Y2dzIKVDFnVVhRZFhPRmREbnVJSzN2ZThuWEg5UndtdDAxNlEvbDdEcS9ZMWxrdzhBcHVEbHI5eHIzaVFicmFjN2VlbgpBcEthR3RVVTJqVEk5VGhacWRTOFI0dmJhU1dmVGZEK0xKUmdoTnpPaGU1VUl4TGtvK2sweTRZTGZ6MzVOY1dQClV6c0NzSjBDZ1lFQTE4Yk52Y0YwbUFHTitwSGNNM2kyc1ZNRmJFN0EwYnlOLzE5MGh6Y0pRYWh4R250TTBqOVkKVUZxU2RaRzBHa203azd0UlFUR2pISkNCc1d6OVoyMEowSkFjSDVvM2xuZk1yaytDUUZXSmk0MmZVdDE1VlRoUwo5d2xJYUJtb1dPbmhsWXlaa0RTMVZXaWNFNys1clIzQ0lxUXhvb0VEMTl4U0I2WmVmV2s1NVo4PQotLS0tLUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=
---------------------------------
</pre>

<pre>
vim $HOME/.kube/config
vim $HOME/.kube/config
vim $HOME/.kube/config
</pre>

<pre>
sudo chown $(id -u):$(id -g) $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
</pre>
----------------------------------------------------------------------------------------------------------------------------

#### проверяем

<pre>
kubectl get nodes
kubectl get nodes
kubectl get nodes

kubectl get nodes

NAME                             STATUS   ROLES           AGE   VERSION
kub1.ru-central1.internal        Ready    <none>          16h   v1.26.1
kub2.ru-central1.internal        Ready    <none>          16h   v1.26.1
kub3.ru-central1.internal        Ready    <none>          16h   v1.26.1
masterkub.ru-central1.internal   Ready    control-plane   20h   v1.26.1
</pre>


#### 5.16)

#### проверяю 

<pre>
----------------------------------------------------------------------------------------------------------------------------
kubectl get nodes

NAME                             STATUS   ROLES           AGE     VERSION
kub1.ru-central1.internal        Ready    <none>          35m     v1.26.1
kub2.ru-central1.internal        Ready    <none>          35s     v1.26.1
kub3.ru-central1.internal        Ready    <none>          16m     v1.26.1
masterkub.ru-central1.internal   Ready    control-plane   4h30m   v1.26.1
----------------------------------------------------------------------------------------------------------------------------
</pre>


#### 5.17)

#### важно! выделил заметка для себя   ---- ноды значения/наименования которых мы ввидем из разных команд

<pre>
----------------------------------------------------------------------------------------------------------------------------
[root@masterkub vorori]# kubectl get nodes
NAME                             STATUS   ROLES           AGE     VERSION
----
kub1.ru-central1.internal        Ready    <none>          37m     v1.26.1
kub2.ru-central1.internal        Ready    <none>          3m16s   v1.26.1
kub3.ru-central1.internal        Ready    <none>          18m     v1.26.1
----
masterkub.ru-central1.internal   Ready    control-plane   4h33m   v1.26.1
[root@masterkub vorori]# kubectl get all -A
NAMESPACE      NAME                                                         READY   STATUS    RESTARTS        AGE
kube-flannel   pod/kube-flannel-ds-77pwz                                    1/1     Running   1 (7m20s ago)   8m8s
kube-flannel   pod/kube-flannel-ds-8nrhx                                    1/1     Running   0               10m
kube-flannel   pod/kube-flannel-ds-d94qd                                    1/1     Running   0               58m
kube-flannel   pod/kube-flannel-ds-pslvk                                    1/1     Running   0               9m7s
kube-system    pod/coredns-787d4945fb-mxvx7                                 1/1     Running   0               65m
kube-system    pod/coredns-787d4945fb-x57tb                                 1/1     Running   0               65m
kube-system    pod/etcd-masterkub.ru-central1.internal                      1/1     Running   0               66m
kube-system    pod/kube-apiserver-masterkub.ru-central1.internal            1/1     Running   0               66m
kube-system    pod/kube-controller-manager-masterkub.ru-central1.internal   1/1     Running   0               66m
kube-system    pod/kube-proxy-25mpq                                         1/1     Running   0               10m
kube-system    pod/kube-proxy-cdhx7                                         1/1     Running   0               65m
kube-system    pod/kube-proxy-ssz9x                                         1/1     Running   0               8m8s
kube-system    pod/kube-proxy-z4prx                                         1/1     Running   0               9m7s
kube-system    pod/kube-scheduler-masterkub.ru-central1.internal            1/1     Running   0               66m

NAMESPACE     NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
default       service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP                  66m
kube-system   service/kube-dns     ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   66m

NAMESPACE      NAME                             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-flannel   daemonset.apps/kube-flannel-ds   4         4         4       4            4           <none>                   58m
kube-system    daemonset.apps/kube-proxy        4         4         4       4            4           kubernetes.io/os=linux   66m

NAMESPACE     NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
kube-system   deployment.apps/coredns   2/2     2            2           66m

NAMESPACE     NAME                                 DESIRED   CURRENT   READY   AGE
kube-system   replicaset.apps/coredns-787d4945fb   2         2         2       65m
----------------------------------------------------------------------------------------------------------------------------
</pre>


<pre>
---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
----------------------------------------------настройка patroni или аналог-------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
</pre>



zalando postgres patroni
https://habr.com/ru/articles/527042/

https://highload.ru/moscow/2019/abstracts/6049




1)
Local Path Provisioner
Local Path Provisioner
Local Path Provisioner
https://www.youtube.com/watch?v=9H0Wp1Xnbf4&ab_channel=%D0%90%D1%80%D1%82%D1%83%D1%80%D0%9A%D1%80%D1%8E%D0%BA%D0%BE%D0%B2
https://www.youtube.com/watch?v=9H0Wp1Xnbf4&ab_channel=%D0%90%D1%80%D1%82%D1%83%D1%80%D0%9A%D1%80%D1%8E%D0%BA%D0%BE%D0%B2
https://www.youtube.com/watch?v=9H0Wp1Xnbf4&ab_channel=%D0%90%D1%80%D1%82%D1%83%D1%80%D0%9A%D1%80%D1%8E%D0%BA%D0%BE%D0%B2



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


2)
Zalando spilo in kubernetes - manifests
Zalando spilo in kubernetes - manifests
Zalando spilo in kubernetes - manifests
https://www.youtube.com/watch?v=fFvOA8UlnrI&ab_channel=%D0%90%D1%80%D1%82%D1%83%D1%80%D0%9A%D1%80%D1%8E%D0%BA%D0%BE%D0%B2
https://www.youtube.com/watch?v=fFvOA8UlnrI&ab_channel=%D0%90%D1%80%D1%82%D1%83%D1%80%D0%9A%D1%80%D1%8E%D0%BA%D0%BE%D0%B2
https://www.youtube.com/watch?v=fFvOA8UlnrI&ab_channel=%D0%90%D1%80%D1%82%D1%83%D1%80%D0%9A%D1%80%D1%8E%D0%BA%D0%BE%D0%B2

3)
Анонс видео: Подробно о создании helm chart.
Анонс видео: Подробно о создании helm chart.
Анонс видео: Подробно о создании helm chart.
https://www.youtube.com/watch?v=bXr7wvMgKkw&ab_channel=%D0%90%D1%80%D1%82%D1%83%D1%80%D0%9A%D1%80%D1%8E%D0%BA%D0%BE%D0%B2
https://www.youtube.com/watch?v=bXr7wvMgKkw&ab_channel=%D0%90%D1%80%D1%82%D1%83%D1%80%D0%9A%D1%80%D1%8E%D0%BA%D0%BE%D0%B2
https://www.youtube.com/watch?v=bXr7wvMgKkw&ab_channel=%D0%90%D1%80%D1%82%D1%83%D1%80%D0%9A%D1%80%D1%8E%D0%BA%D0%BE%D0%B2


https://www.youtube.com/watch?v=vv8SSYITzPE&list=PLmxqUDFl0XM7e0d0ixZ82zlcBprpMfEpk&ab_channel=%D0%90%D1%80%D1%82%D1%83%D1%80%D0%9A%D1%80%D1%8E%D0%BA%D0%BE%D0%B2
https://www.youtube.com/watch?v=vv8SSYITzPE&list=PLmxqUDFl0XM7e0d0ixZ82zlcBprpMfEpk&ab_channel=%D0%90%D1%80%D1%82%D1%83%D1%80%D0%9A%D1%80%D1%8E%D0%BA%D0%BE%D0%B2
https://www.youtube.com/watch?v=vv8SSYITzPE&list=PLmxqUDFl0XM7e0d0ixZ82zlcBprpMfEpk&ab_channel=%D0%90%D1%80%D1%82%D1%83%D1%80%D0%9A%D1%80%D1%8E%D0%BA%D0%BE%D0%B2

4)
Zalando postgres-operator [01] Артур Крюков
https://www.youtube.com/watch?v=1eJ8njJqIS4&t=784s&ab_channel=%D0%90%D1%80%D1%82%D1%83%D1%80%D0%9A%D1%80%D1%8E%D0%BA%D0%BE%D0%B2
https://www.youtube.com/watch?v=1eJ8njJqIS4&t=784s&ab_channel=%D0%90%D1%80%D1%82%D1%83%D1%80%D0%9A%D1%80%D1%8E%D0%BA%D0%BE%D0%B2
https://www.youtube.com/watch?v=1eJ8njJqIS4&t=784s&ab_channel=%D0%90%D1%80%D1%82%D1%83%D1%80%D0%9A%D1%80%D1%8E%D0%BA%D0%BE%D0%B2



https://www.youtube.com/watch?v=iruaCgeG7qs&ab_channel=Apprenda
https://www.youtube.com/watch?v=iruaCgeG7qs&ab_channel=Apprenda



































































#### 1)

#### добавил метки 

<pre>
kubectl label nodes masterkub.ru-central1.internal disktype=citusmaster
kubectl label nodes kub1.ru-central1.internal disktype=citusworker1
kubectl label nodes kub2.ru-central1.internal disktype=citusworker2
kubectl label nodes kub3.ru-central1.internal disktype=citusworker3

citusmaster
citusworker1
citusworker2
citusworker3
</pre>

#### 1.2)

#### проверяю

<pre>
[root@masterkub vorori]# kubectl get nodes --show-labels
NAME                             STATUS   ROLES           AGE     VERSION   LABELS
kub1.ru-central1.internal        Ready    <none>          2d6h    v1.26.1   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,disktype=citusworker1,kubernetes.io/arch=amd64,kubernetes.io/hostname=kub1.ru-central1.internal,kubernetes.io/os=linux
kub2.ru-central1.internal        Ready    <none>          2d5h    v1.26.1   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,disktype=citusworker2,kubernetes.io/arch=amd64,kubernetes.io/hostname=kub2.ru-central1.internal,kubernetes.io/os=linux
kub3.ru-central1.internal        Ready    <none>          2d5h    v1.26.1   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,disktype=citusworker3,kubernetes.io/arch=amd64,kubernetes.io/hostname=kub3.ru-central1.internal,kubernetes.io/os=linux
masterkub.ru-central1.internal   Ready    control-plane   2d10h   v1.26.1   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,disktype=citusmaster,kubernetes.io/arch=amd64,kubernetes.io/hostname=masterkub.ru-central1.internal,kubernetes.io/os=
</pre>

#### 1.3)

#### создал директорию для мастера и worker

<pre>
mkdir /var/pgsql-volume-master
mkdir /var/pgsql-volume
</pre>


#### 1.4)

#### создал pv

<pre>
vim /tmp/citus/mypv.yaml

apiVersion: v1
kind: PersistentVolume
metadata:
  name: storage-citusmaster
  labels:
    storage: basemain1citusmaster1
spec:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 10Gi
  local:
    path: /var/pgsql-volume-master
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: disktype
          operator: In
          values:
          - citusworker1
  persistentVolumeReclaimPolicy: Retain
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: storage-citusworker1
  labels:
    storage: baserepl1citusworker1
spec:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 10Gi
  local:
    path: /var/pgsql-volume
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: disktype
          operator: In
          values:
          - citusworker1
  persistentVolumeReclaimPolicy: Retain
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: storage-citusworker2
  labels:
    storage: baserepl2citusworker2
spec:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 10Gi
  local:
    path: /var/pgsql-volume
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: disktype
          operator: In
          values:
          - citusworker2
  persistentVolumeReclaimPolicy: Retain
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: storage-citusworker3
  labels:
    storage: baserepl3citusworker3
spec:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 10Gi
  local:
    path: /var/pgsql-volume
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: disktype
          operator: In
          values:
          - citusworker3
  persistentVolumeReclaimPolicy: Retain

</pre>

#### 1.5)

#### проверяем 

<pre>
[root@masterkub citus]# kubectl apply -f /tmp/citus/mypv.yaml
persistentvolume/storage-citusworker1 created
[root@masterkub citus]# kubectl get pv
NAME                   CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
storage-citusworker1   10Gi       RWO            Retain           Available                                   12s                         44s
</pre>


#### 1.6)

#### создал secrets

<pre>
vim /tmp/citus/secrets.yaml


apiVersion: v1
kind: Secret
metadata:
  name: citus-secrets
type: Opaque
data:
  password: MjMzODQ4NA==
</pre>

<pre>
kubectl create -f /tmp/citus/secrets.yaml

[root@masterkub citus]# [root@masterkub citus]# kubectl create -f /tmp/citus/secrets.yaml
secret/citus-secrets created
</pre>


#### 1.7)

#### создал PVC citus-master

<pre>
https://coffee-web.ru/blog/getting-started-with-kubernetes-persistent-volumes/
Создание заявки на постоянный том
Создайте спецификацию PVC, открыв текстовый редактор и скопировав приведенный ниже код YA

vim /tmp/citus/pvcmaster.yaml

kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: citus-master-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  volumeName: storage-citusmaster
</pre>

#### 1.8)

#### проверяем 

<pre>
kubectl apply -f /tmp/citus/pvcmaster.yaml
kubectl get pvc -o wide
[root@masterkub citus]# [root@masterkub citus]# kubectl apply -f /tmp/citus/pvcmaster.yaml
persistentvolumeclaim/citus-master-pvc created
[root@masterkub citus]# kubectl get pvc -o wide
NAME               STATUS   VOLUME                 CAPACITY   ACCESS MODES   STORAGECLASS   AGE   VOLUMEMODE
citus-master-pvc   Bound    storage-citusworker1   10Gi       RWO                           12s   Filesystem
</pre>


#### 1.9)

#### создаю master

<pre>
vim /tmp/citus/master.yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: citus-master-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
---
apiVersion: v1
kind: Service
metadata:
  name: citus-master
  labels:
    app: citus-master
spec:
  selector:
    app: citus-master
  type: NodePort
  ports:
  - port: 5432
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: citus-master
spec:
  selector:
    matchLabels:
      app: citus-master
  replicas: 1
  template:
    metadata:
      labels:
        app: citus-master
    spec:
      containers:
      - name: citus
        image: citusdata/citus:7.3.0
        ports:
        - containerPort: 5432
        env:
        - name: PGDATA
          value: /var/lib/postgresql/data/pgdata
        - name: PGPASSWORD
          valueFrom:
            secretKeyRef:
              name: citus-secrets
              key: password
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: citus-secrets
              key: password
        volumeMounts:
        - name: storage
          mountPath: /var/lib/postgresql/data
        livenessProbe:
          exec:
            command:
            - ./pg_healthcheck
          initialDelaySeconds: 60
      volumes:
        - name: storage
          persistentVolumeClaim:
            claimName: citus-master-pvc
</pre>		

#### 1.10)

#### проверяю

<pre>
kubectl apply -f /tmp/citus/master.yaml
[root@masterkub citus]# kubectl get pods
NAME                            READY   STATUS    RESTARTS   AGE
citus-master-5bff7bc99c-kd89l   1/1     Running   0          41s
</pre>

#### 1.11)

<pre>
[root@kub1 pgsql-volume-master]# ls -l /var/pgsql-volume-master/pgdata
total 56
drwx------. 6 polkitd input    64 Jul 15 21:13 base
drwx------. 2 polkitd input  4096 Jul 15 21:14 global
drwx------. 2 polkitd input     6 Jul 15 21:13 pg_commit_ts
drwx------. 2 polkitd input     6 Jul 15 21:13 pg_dynshmem
drwx------. 3 polkitd input    20 Jul 15 21:13 pg_foreign_file
-rw-------. 1 polkitd input  4535 Jul 15 21:13 pg_hba.conf
-rw-------. 1 polkitd input  1636 Jul 15 21:13 pg_ident.conf
drwx------. 4 polkitd input    68 Jul 15 21:13 pg_logical
drwx------. 4 polkitd input    36 Jul 15 21:13 pg_multixact
drwx------. 2 polkitd input    18 Jul 15 21:13 pg_notify
drwx------. 2 polkitd input     6 Jul 15 21:13 pg_replslot
drwx------. 2 polkitd input     6 Jul 15 21:13 pg_serial
drwx------. 2 polkitd input     6 Jul 15 21:13 pg_snapshots
drwx------. 2 polkitd input     6 Jul 15 21:13 pg_stat
drwx------. 2 polkitd input    63 Jul 15 21:17 pg_stat_tmp
drwx------. 2 polkitd input    18 Jul 15 21:13 pg_subtrans
drwx------. 2 polkitd input     6 Jul 15 21:13 pg_tblspc
drwx------. 2 polkitd input     6 Jul 15 21:13 pg_twophase
-rw-------. 1 polkitd input     3 Jul 15 21:13 PG_VERSION
drwx------. 3 polkitd input    60 Jul 15 21:13 pg_wal
drwx------. 2 polkitd input    18 Jul 15 21:13 pg_xact
-rw-------. 1 polkitd input    88 Jul 15 21:13 postgresql.auto.conf
-rw-------. 1 polkitd input 22762 Jul 15 21:13 postgresql.conf
-rw-------. 1 polkitd input    36 Jul 15 21:13 postmaster.opts
-rw-------. 1 polkitd input   101 Jul 15 21:13 postmaster.pid
</pre>


#### 1.12)

#### создаю worker

<pre>
vim /tmp/citus/worker.yaml

apiVersion: v1
kind: Service
metadata:
  name: citus-workers
  labels:
    app: citus-workers
spec:
  selector:
    app: citus-workers
  clusterIP: None
  ports:
  - port: 5432
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: citus-worker
spec:
  selector:
    matchLabels:
      app: citus-workers
  serviceName: citus-workers
  replicas: 3
  template:
    metadata:
      labels:
        app: citus-workers
    spec:
      containers:
      - name: citus-worker
        image: citusdata/citus:7.3.0
        lifecycle:
          postStart:
            exec:
              command: 
              - /bin/sh
              - -c
              - if [ ${POD_IP} ]; then psql --host=citus-master --username=postgres --command="SELECT * from master_add_node('${HOSTNAME}.citus-workers', 5432);" ; fi
        ports:
        - containerPort: 5432
        env:
        - name: POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        - name: PGPASSWORD
          valueFrom:
            secretKeyRef:
              name: citus-secrets
              key: password
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: citus-secrets
              key: password
        - name: PGDATA
          value: /var/lib/postgresql/data/pgdata
        volumeMounts:
        - name: storage
          mountPath: /var/lib/postgresql/data
        livenessProbe:
          exec:
            command:
            - ./pg_healthcheck
          initialDelaySeconds: 60
  volumeClaimTemplates:
  - metadata:
      name: storage
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 10Gi
</pre>


#### 1.13)

#### проверяю

<pre>
kubectl apply -f /tmp/citus/worker.yaml
[root@masterkub citus]# [root@masterkub citus]# kubectl apply -f /tmp/citus/worker.yaml
service/citus-workers created
statefulset.apps/citus-worker created
[root@masterkub citus]# kubectl get pods
NAME                            READY   STATUS    RESTARTS   AGE
citus-master-5bff7bc99c-kd89l   1/1     Running   0          2m32s
citus-worker-0                  1/1     Running   0          14s
citus-worker-1                  1/1     Running   0          11s
citus-worker-2                  1/1     Running   0          9s
</pre>

#### 1.14)

#### проверяю

<pre>
kubectl exec -it pod/citus-master-5bff7bc99c-kd89l -- bash
psql -U postgres
postgres=# SELECT * FROM master_get_active_worker_nodes();
          node_name           | node_port
------------------------------+-----------
 citus-worker-2.citus-workers |      5432
 citus-worker-0.citus-workers |      5432
 citus-worker-1.citus-workers |      5432
(3 rows)
</pre>



#### 1.15)

#### настройка подготовка к заливке данных

<pre>
установил  клиент psql
yum install postgresql

открыл доступ c 10.129.0.34 
vim /var/pgsql-volume-master/pgdata/pg_hba.conf

host    all             postgres           10.129.0.34/32            trust

применил наcтройки:
select pg_reload_conf();

прокинул подключения контенера наружу
kubectl port-forward pod/citus-master-5bff7bc99c-kd89l 5432:5432

подключился c kub1.ru-central1.internal
psql -U postgres -h 127.0.0.1

create database test;
CREATE EXTENSION citus;
</pre>

#### 1.16)

#### создаю базу на всех воркерах

<pre>
kubectl exec -it pod/citus-worker-0 -- psql -U postgres -c 'create database test;'
kubectl exec -it pod/citus-worker-1 -- psql -U postgres -c 'create database test;'
kubectl exec -it pod/citus-worker-2 -- psql -U postgres -c 'create database test;'

kubectl exec -it pod/citus-worker-0 -- psql -U postgres -d test -c 'CREATE EXTENSION citus;'
kubectl exec -it pod/citus-worker-1 -- psql -U postgres -d test -c 'CREATE EXTENSION citus;'
kubectl exec -it pod/citus-worker-2 -- psql -U postgres -d test -c 'CREATE EXTENSION citus;'
</pre>

#### 1.17)

#### подключаю ноды

<pre>
sudo -i -u postgres psql -c "SELECT * FROM master_add_node('citus-worker-1.citus-workers', 5432);"
sudo -i -u postgres psql -c "SELECT * FROM master_add_node('citus-worker-0.citus-workers', 5432);"
sudo -i -u postgres psql -c "SELECT * FROM master_add_node('citus-worker-2.citus-workers', 5432);"
</pre>

#### 1.18)

#### проверяю

<pre>
test=# SELECT * FROM master_get_active_worker_nodes();
          node_name           | node_port
------------------------------+-----------
 citus-worker-2.citus-workers |      5432
 citus-worker-0.citus-workers |      5432
 citus-worker-1.citus-workers |      5432
(3 rows)
</pre>



#### 1.19)

#### создание табл и заливка данных 

<pre>
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

#### 1.20)

#### по unique_key

<pre>
SELECT create_distributed_table('taxi_trips', 'unique_key');
SELECT create_distributed_table('taxi_trips', 'unique_key');
SELECT create_distributed_table('taxi_trips', 'unique_key');
</pre>

#### 1.21)

#### подключил бакет через s3fs-fuse

<pre>
mkdir /tmp/taxi
cd /tmp/taxi
su - postgres
добавляю ранее созданный индификатор,двоеточие и сервисный ключ
/home/postgres/.passwd-s3fs
echo YCAJEUqYD2tU3jgU2_jXoYImx:YCPQB68gMTtEmWhj-UQNp7HvY85KZFkBYdWL1lHP > /home/postgres/.passwd-s3fs
cat /home/postgres/.passwd-s3fs
chmod 600 /home/postgres/.passwd-s3fs
s3fs myotus /tmp/taxi -o passwd_file=/home/postgres/.passwd-s3fs -o url=https://storage.yandexcloud.net -o use_path_request_style -o dbglevel=info -f -o curldbg
chown postgres:postgres /tmp/taxi -R
</pre>

#### 1.22)

#### вливаю данные

<pre>
kubectl port-forward pod/citus-master-5bff7bc99c-kd89l 5432:5432
for f in *.csv*; do psql -U postgres -p 5432 -h localhost -d test -c "\\COPY taxi_trips FROM PROGRAM 'cat $f' CSV HEADER"; done

или

\COPY taxi_trips FROM '/tmp/taxi2/taxi.csv.000000000000' DELIMITER ',' CSV HEADER;
\COPY taxi_trips FROM '/tmp/taxi2/taxi.csv.000000000001' DELIMITER ',' CSV HEADER;
\COPY taxi_trips FROM '/tmp/taxi2/taxi.csv.000000000002' DELIMITER ',' CSV HEADER;
\COPY taxi_trips FROM '/tmp/taxi2/taxi.csv.000000000003' DELIMITER ',' CSV HEADER;
\COPY taxi_trips FROM '/tmp/taxi2/taxi.csv.000000000004' DELIMITER ',' CSV HEADER;
\COPY taxi_trips FROM '/tmp/taxi2/taxi.csv.000000000005' DELIMITER ',' CSV HEADER;
\COPY taxi_trips FROM '/tmp/taxi2/taxi.csv.000000000006' DELIMITER ',' CSV HEADER;
\COPY taxi_trips FROM '/tmp/taxi2/taxi.csv.000000000007' DELIMITER ',' CSV HEADER;
\COPY taxi_trips FROM '/tmp/taxi2/taxi.csv.000000000008' DELIMITER ',' CSV HEADER;
\COPY taxi_trips FROM '/tmp/taxi2/taxi.csv.000000000009' DELIMITER ',' CSV HEADER;
\COPY taxi_trips FROM '/tmp/taxi2/taxi.csv.000000000010' DELIMITER ',' CSV HEADER;
\COPY taxi_trips FROM '/tmp/taxi2/taxi.csv.000000000011' DELIMITER ',' CSV HEADER;
\COPY taxi_trips FROM '/tmp/taxi2/taxi.csv.000000000012' DELIMITER ',' CSV HEADER;
\COPY taxi_trips FROM '/tmp/taxi2/taxi.csv.000000000013' DELIMITER ',' CSV HEADER;
\COPY taxi_trips FROM '/tmp/taxi2/taxi.csv.000000000014' DELIMITER ',' CSV HEADER;
\COPY taxi_trips FROM '/tmp/taxi2/taxi.csv.000000000015' DELIMITER ',' CSV HEADER;
\COPY taxi_trips FROM '/tmp/taxi2/taxi.csv.000000000016' DELIMITER ',' CSV HEADER;
\COPY taxi_trips FROM '/tmp/taxi2/taxi.csv.000000000017' DELIMITER ',' CSV HEADER;
\COPY taxi_trips FROM '/tmp/taxi2/taxi.csv.000000000018' DELIMITER ',' CSV HEADER;
\COPY taxi_trips FROM '/tmp/taxi2/taxi.csv.000000000019' DELIMITER ',' CSV HEADER;
\COPY taxi_trips FROM '/tmp/taxi2/taxi.csv.000000000020' DELIMITER ',' CSV HEADER;
\COPY taxi_trips FROM '/tmp/taxi2/taxi.csv.000000000021' DELIMITER ',' CSV HEADER;
\COPY taxi_trips FROM '/tmp/taxi2/taxi.csv.000000000022' DELIMITER ',' CSV HEADER;
\COPY taxi_trips FROM '/tmp/taxi2/taxi.csv.000000000023' DELIMITER ',' CSV HEADER;
\COPY taxi_trips FROM '/tmp/taxi2/taxi.csv.000000000024' DELIMITER ',' CSV HEADER;
\COPY taxi_trips FROM '/tmp/taxi2/taxi.csv.000000000025' DELIMITER ',' CSV HEADER;
</pre>


<pre>
---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------итого по запросам ------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
</pre>


<pre>
---------------------------------------------------------------------------------------------------------------------------------------
EXPLAIN(ANALYZE) SELECT payment_type, round(sum(tips))/round(sum(trip_total)*100) + 0 as tips_percent, count(*) as c 
FROM taxi_trips
group by payment_type
order by c;


на citus
Execution time: 11738.133 ms

на gp 
Execution time: 36975.560 ms

на Postgre
Execution Time: 208397.652 ms
---------------------------------------------------------------------------------------------------------------------------------------
</pre>

<pre>
---------------------------------------------------------------------------------------------------------------------------------------
тестовая генерация данных:
EXPLAIN(ANALYZE) create table t14 as select generate_series(1,1000000) as colA distributed by (colA);

на citus
Execution time: 2435.434 ms

на gp 
Execution time: 804.079 ms

на ванильном Postgre
Execution time: 3.981 sec
---------------------------------------------------------------------------------------------------------------------------------------
</pre>

<pre>
---------------------------------------------------------------------------------------------------------------------------------------
запрос:
EXPLAIN(ANALYZE) select count(*) from t14;

на citus
Execution time: 98.504 ms

на gp
Execution time: 182.096 ms

на ванильном Postgre
Execution Time: 114.369 ms
---------------------------------------------------------------------------------------------------------------------------------------
</pre>

<pre>
---------------------------------------------------------------------------------------------------------------------------------------
запрос:
EXPLAIN(ANALYZE) select sum(colA) from t14;

на citus
Execution time: 67.226 ms

на gp
Execution time: 79.044 ms

на ванильном Postgre
Execution Time: 102.372 ms
---------------------------------------------------------------------------------------------------------------------------------------
</pre>

<pre>
---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------трудности---------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
</pre>

<pre>
когда настраивал кластер k8s проблем было очень много. универсально рецепта нет нигде в каждой версии свои приколы почти на каждом шаге по созданию кластера вываливались 
ошибки причем отлаживал паралельно разные версии на своих vm в соей сетке и  в облаках ... "баги" при инициализации кластера это отдельная тема заметил что в яндекс оболаке
было больше напрягов по траблошутингу. Эта домашка далась очень сложно отступать было нельзя, диплом планировал связать с k8s.
большое количество времени потратил на осмысление материала и на практике оттачивал многие моменты чтобы было хоть какое понимание как же оно всетаки работает.
паралельно собрал свой локальный домашний кластер правда его надо допилить хорошенько напильником...(всего попыток настройки кластера k8s инсталяций было около 4х) 
на яндекс в облако когда заходил в тупик убивл виртуалки и заново все разворачивал после того как наковырял исправления ошибок проблем на последующих шагах было еше больше....
уж очень плохая реализация "снапшетов" в яндекс облаке которая прям подбешивает их впринцепи можно таки сказать и нет(. 
Итого неделька в режиме вампир по ночам нон стоп первое сильное знакомство с Kubernetes удалось)))
</pre>

<pre>
ошибок было намного больше вот некоторые из них:
container
-----------------------------------------------
Jul 13 06:55:55 masterkub kubelet: E0713 06:55:55.831658    1708 run.go:74] "command failed" err="failed to validate kubelet flags: 
the container runtime endpoint address was not specified or empty, use --container-runtime-endpoint to set"
-----------------------------------------------

не поднимался системный pod
-----------------------------------------------
kube-system   kube-flannel-ds-amd64-42rl7            0/1       CrashLoopBackOff
-----------------------------------------------

проблема с PersistentVolumeClaims
-----------------------------------------------
0/4 nodes are available: pod has unbound immediate PersistentVolumeClaims. preemption: 0/4 nodes are available: 4 No preemption victims found for incoming pod..
-----------------------------------------------
</pre>





<pre>
---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------вспомогательные команды-------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
</pre>


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
------------
------------
Это предоставит информацию обо всех модулях, развертываниях, службах и заданиях в пространстве имен.

kubectl get pods,services,deployments,jobs
kubectl get pods,services,deployments,jobs
kubectl get pods,services,deployments,jobs

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
