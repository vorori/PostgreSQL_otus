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



мои заметки к диплому выбираем statesfuleset чтобы сохранять измнения контенеров с бд
replicas 3   --- создастца один мастер под и 2 слейва



---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------ПУНКТЫ КОТОРЫЕ ТРЕБУЕТСЯ ИЗУЧИТЬ!!!!------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
0)
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



дополнительно для информации:
https://www.youtube.com/watch?v=iruaCgeG7qs&ab_channel=Apprenda
https://www.youtube.com/watch?v=iruaCgeG7qs&ab_channel=Apprenda






<pre>
---------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
----------------------------------------------настройка patroni часть 1 подготовка-----------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
</pre>



1)
https://github.com/BigKAA/youtube/tree/38f295485674147bc484c2183625059987a46013/base/local-path-provisioner

установить настроить local-path-provisioner

скачать манифест редактируем по своим настройкам:
https://github.com/BigKAA/youtube/blob/38f295485674147bc484c2183625059987a46013/base/local-path-provisioner/manifests/00-local-path-storage.yaml

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

https://github.com/zalando/spilo
https://github.com/BigKAA/youtube/blob/38f295485674147bc484c2183625059987a46013/base/spilo/manifests/spilo_kubernetes.yaml
https://github.com/BigKAA/youtube/blob/38f295485674147bc484c2183625059987a46013/base/spilo/Spilo-manual.md
https://github.com/zalando/spilo/blob/master/ENVIRONMENT.rst



---------------------------------------------------------------------------------------------------------------------------------------
----------------------------------------------Конфигурационные параметры скрипта START-------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------
#Конфигурационные параметры скрипта
#Параметры скрипта поместив в отдельный ConfigMap

vim /data/spilo_backup-script.yaml
kubectl apply -f /data/spilo_backup-script.yaml

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
  WALG_ALIVE_CHECK_INTERVAL: "5m"
  WALE_BINARY: "wal-g"
  WALG_FILE_PREFIX: "/data/pg_wal"
  WALE_ENV_DIR: "/config"

---------------------------------------------------------------------------------------------------------------------------------------
----------------------------------------------Конфигурационные параметры скрипта END---------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------

#проверяем
kubectl get ConfigMap
kubectl delete configmap backup-script



---------------------------------------------------------------------------------------------------------------------------------------
----------------------------------------------Конфигурационные параметры zalandopatroni START------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------

vim /data/spilo_kubernetes.yaml


apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: &cluster_name zalandopatroni01
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
              initdb:
                - auth-host: md5
                - auth-local: md5
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
          value: "/data/pg_wal"
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
            claimName: zalandopatroni01-backup
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
          storage: 2Gi
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
          storage: 2Gi
---
apiVersion: v1
kind: Endpoints
metadata:
  name: &cluster_name zalandopatroni01
  labels:
    application: spilo
    spilo-cluster: *cluster_name
subsets: []

---
apiVersion: v1
kind: Service
metadata:
  name: &cluster_name zalandopatroni01
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
  name: zalandopatroni01-config
  labels:
    application: spilo
    spilo-cluster: zalandopatroni01
spec:
  clusterIP: None

---
apiVersion: v1
kind: Secret
metadata:
  name: &cluster_name zalandopatroni01
  labels:
    application: spilo
    spilo-cluster: *cluster_name
type: Opaque
stringData:
  superuser-password: password
  replication-password: password
  admin-password: password

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
  WALG_ALIVE_CHECK_INTERVAL: "5m"
  WALE_BINARY: "wal-g"
  WALG_FILE_PREFIX: "/data/pg_wal"
  WALE_ENV_DIR: "/config"


---------------------------------------------------------------------------------------------------------------------------------------
----------------------------------------------Конфигурационные параметры zalandopatroni END--------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------





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



#начинаем создание нашего кластера
#создаем namespase spilo
kubectl create ns spilo
kubectl create ns spilo

### Это предоставит информацию о пространствах имен
kubectl get namespace
kubectl get namespace

kubectl get all -A
kubectl get all -A

#запускаю наш манифест 
kubectl -n spilo apply -f /data/spilo_kubernetes.yaml
kubectl -n spilo apply -f /data/spilo_kubernetes.yaml

#проверяю что все ок и что все запустилось все под создались и работают
kubectl get pods --namespace spilo
kubectl get pods --namespace spilo

----------
[root@masterkub vorori]# kubectl get pods --namespace spilo
NAME                 READY   STATUS    RESTARTS   AGE
zalandopatroni01-0   1/1     Running   0          7m15s
zalandopatroni01-1   1/1     Running   0          7m15s
zalandopatroni01-2   1/1     Running   0          7m15s
----------

#проверяю что создались сервисы
kubectl get services --namespace spilo
kubectl get services --namespace spilo

----------
[root@masterkub vorori]# kubectl get services --namespace spilo
NAME                      TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
zalandopatroni01          ClusterIP   10.105.5.80   <none>        5432/TCP   26m
zalandopatroni01-config   ClusterIP   None          <none>        <none>     26m
----------

#проверяю что все ок и что все запустилось с дисками pvc
kubectl get pvc --namespace spilo
kubectl get pvc --namespace spilo

----------
[root@masterkub vorori]# kubectl get pvc --namespace spilo
NAME                        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
backup-zalandopatroni01-0   Bound    pvc-27ed630b-b7df-4a04-802b-67e547bfc272   2Gi        RWO            local-path     11m
backup-zalandopatroni01-1   Bound    pvc-afa11811-90df-4a0d-ad2e-adaf27126c12   2Gi        RWO            local-path     11m
backup-zalandopatroni01-2   Bound    pvc-9b76b757-e3e7-4581-9a5c-4e510e3bd898   2Gi        RWO            local-path     11m
pgdata-zalandopatroni01-0   Bound    pvc-dc04c3ee-aaf9-4698-be1a-64bcbd66119a   2Gi        RWO            local-path     11m
pgdata-zalandopatroni01-1   Bound    pvc-58ae2079-216f-4353-8edb-fbe930ebeeba   2Gi        RWO            local-path     11m
pgdata-zalandopatroni01-2   Bound    pvc-eddb585f-e9af-48ee-aab5-aaeb971716f4   2Gi        RWO            local-path     11m
----------

#проверяю что все ок на всех обектах кластера 
----------
[root@masterkub vorori]# kubectl get all -A
NAMESPACE            NAME                                                         READY   STATUS    RESTARTS      AGE
kube-flannel         pod/kube-flannel-ds-77pwz                                    1/1     Running   4 (29m ago)   3d22h
kube-flannel         pod/kube-flannel-ds-8nrhx                                    1/1     Running   2 (31m ago)   3d22h
kube-flannel         pod/kube-flannel-ds-d94qd                                    1/1     Running   2 (31m ago)   3d22h
kube-flannel         pod/kube-flannel-ds-pslvk                                    1/1     Running   2 (31m ago)   3d22h
kube-system          pod/coredns-787d4945fb-mxvx7                                 1/1     Running   2 (31m ago)   3d23h
kube-system          pod/coredns-787d4945fb-x57tb                                 1/1     Running   2 (31m ago)   3d23h
kube-system          pod/etcd-masterkub.ru-central1.internal                      1/1     Running   2 (31m ago)   3d23h
kube-system          pod/kube-apiserver-masterkub.ru-central1.internal            1/1     Running   2 (31m ago)   3d23h
kube-system          pod/kube-controller-manager-masterkub.ru-central1.internal   1/1     Running   2 (31m ago)   3d23h
kube-system          pod/kube-proxy-25mpq                                         1/1     Running   2 (31m ago)   3d22h
kube-system          pod/kube-proxy-cdhx7                                         1/1     Running   2 (31m ago)   3d23h
kube-system          pod/kube-proxy-ssz9x                                         1/1     Running   2 (31m ago)   3d22h
kube-system          pod/kube-proxy-z4prx                                         1/1     Running   2 (31m ago)   3d22h
kube-system          pod/kube-scheduler-masterkub.ru-central1.internal            1/1     Running   2 (31m ago)   3d23h
local-path-storage   pod/local-path-provisioner-7f8667b75c-swvwb                  1/1     Running   1 (31m ago)   18h
spilo                pod/zalandopatroni01-0                                       1/1     Running   0             8m53s
spilo                pod/zalandopatroni01-1                                       1/1     Running   0             8m53s
spilo                pod/zalandopatroni01-2                                       1/1     Running   0             8m53s

NAMESPACE     NAME                              TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)                  AGE
default       service/kubernetes                ClusterIP   10.96.0.1     <none>        443/TCP                  3d23h
kube-system   service/kube-dns                  ClusterIP   10.96.0.10    <none>        53/UDP,53/TCP,9153/TCP   3d23h
spilo         service/zalandopatroni01          ClusterIP   10.105.5.80   <none>        5432/TCP                 8m53s
spilo         service/zalandopatroni01-config   ClusterIP   None          <none>        <none>                   8m53s

NAMESPACE      NAME                             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-flannel   daemonset.apps/kube-flannel-ds   4         4         4       4            4           <none>                   3d22h
kube-system    daemonset.apps/kube-proxy        4         4         4       4            4           kubernetes.io/os=linux   3d23h

NAMESPACE            NAME                                     READY   UP-TO-DATE   AVAILABLE   AGE
kube-system          deployment.apps/coredns                  2/2     2            2           3d23h
local-path-storage   deployment.apps/local-path-provisioner   1/1     1            1           18h

NAMESPACE            NAME                                                DESIRED   CURRENT   READY   AGE
kube-system          replicaset.apps/coredns-787d4945fb                  2         2         2       3d23h
local-path-storage   replicaset.apps/local-path-provisioner-7f8667b75c   1         1         1       18h

NAMESPACE   NAME                                READY   AGE
spilo       statefulset.apps/zalandopatroni01   3/3     8m53s
----------



### смотрим на кворум кто мастер из самого пода pod/zalandopatroni01-0 
kubectl -n spilo exec -it pod/zalandopatroni01-0  -- patronictl list
kubectl -n spilo exec -it pod/zalandopatroni01-0  -- patronictl list

----------
[root@masterkub vorori]# kubectl -n spilo exec -it pod/zalandopatroni01-0  -- patronictl list
+ Cluster: zalandopatroni01 ------+---------+---------+----+-----------+
| Member             | Host       | Role    | State   | TL | Lag in MB |
+--------------------+------------+---------+---------+----+-----------+
| zalandopatroni01-0 | 10.244.1.4 | Replica | running |  1 |         0 |
| zalandopatroni01-1 | 10.244.2.4 | Leader  | running |  1 |           |
| zalandopatroni01-2 | 10.244.3.6 | Replica | running |  1 |         0 |
+--------------------+------------+---------+---------+----+-----------+
----------

### единоразово смотрим логи pod видм кто у нас матер
kubectl logs --namespace spilo pod/zalandopatroni01-0 
kubectl logs --namespace spilo pod/zalandopatroni01-1 
kubectl logs --namespace spilo pod/zalandopatroni01-2


----------
2023-08-15 07:33:41,879 INFO: no action. I am (zalandopatroni01-0), a secondary, and following a leader (zalandopatroni01-1)
2023-08-15 07:33:49,325 INFO: no action. I am (zalandopatroni01-0), a secondary, and following a leader (zalandopatroni01-1)
2023-08-15 07:33:59,333 INFO: no action. I am (zalandopatroni01-0), a secondary, and following a leader (zalandopatroni01-1)


2023-08-15 07:51:59,259 INFO: no action. I am (zalandopatroni01-1), the leader with the lock
2023-08-15 07:52:09,261 INFO: no action. I am (zalandopatroni01-1), the leader with the lock
2023-08-15 07:52:19,259 INFO: no action. I am (zalandopatroni01-1), the leader with the lock


2023-08-15 07:55:09,266 INFO: no action. I am (zalandopatroni01-2), a secondary, and following a leader (zalandopatroni01-1)
2023-08-15 07:55:19,269 INFO: no action. I am (zalandopatroni01-2), a secondary, and following a leader (zalandopatroni01-1)
2023-08-15 07:55:29,265 INFO: no action. I am (zalandopatroni01-2), a secondary, and following a leader (zalandopatroni01-1)
2023-08-15 07:55:39,265 INFO: no action. I am (zalandopatroni01-2), a secondary, and following a leader (zalandopatroni01-1)
----------

### прикручиваем сервис который будет служить единой точкой входа 

#проверяем что сервис создан
kubectl get services --namespace spilo
kubectl get services --namespace spilo

#видим ip адрес сервиса но нам нужно сделать так чтобы сервис ссылался на master ноду проверим как работает оно сейчас
[root@masterkub vorori]# kubectl get services --namespace spilo
NAME                      TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
zalandopatroni01          ClusterIP   10.105.5.80   <none>        5432/TCP   40m
zalandopatroni01-config   ClusterIP   None          <none>        <none>     40m

#устанавливаю клиента psql
yum install postgresql
yum install postgresql

#пытаемся подключиться получаем ошибку это значинт надо моменять конфиг pg_hba.conf
psql -U postgres -h 10.105.5.80
psql -U postgres -h 10.105.5.80

--------------------
psql: FATAL:  password authentication failed for user "postgres"
FATAL:  pg_hba.conf rejects connection for host "10.244.0.0", user "postgres", database "postgres", no encryption
--------------------

#Наборы скриптов для patroni находятся тут
kubectl -n spilo exec -it pod/zalandopatroni01-1 -- ls /scripts/
kubectl -n spilo exec -it pod/zalandopatroni01-1 -- ls /scripts/

#удалил из pg_hba.conf запрешающее правило reject ssl
pg_hba.conf

#из логов мы знаем кто мастер подключимся напрямую к этой ноде мастера чтобы добавить разрешаюшие правила в pg_hba.conf
kubectl -n spilo exec -it pod/zalandopatroni01-1  -- patronictl list
kubectl -n spilo exec -it pod/zalandopatroni01-1  -- patronictl list

#подключаемся к лидеру 
kubectl -n spilo exec -it pod/zalandopatroni01-1  -- bash
kubectl -n spilo exec -it pod/zalandopatroni01-1  -- bash

---------------
[root@masterkub vorori]# kubectl -n spilo exec -it pod/zalandopatroni01-1  -- bash

 ____        _ _
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

run: /etc/service/cron: (pid 37) 3298s
run: /etc/service/patroni: (pid 36) 3298s
run: /etc/service/pgqd: (pid 38) 3298s
---------------

#нашли файл с конфигурацией patroni
---------------
root@zalandopatroni01-1:/home/postgres# pwd
/home/postgres

root@zalandopatroni01-1:/home/postgres# ls -l
total 4
lrwxrwxrwx. 1 root     root   8 Mar 10 05:53 etc -> /run/etc
drwxrwxrwx. 3 root     root  20 Aug 15 07:32 pgdata
-rw-rw-r--. 1 postgres root 157 Mar 10 05:37 pgq_ticker.ini
lrwxrwxrwx. 1 root     root  17 Mar 10 05:53 postgres.yml -> /run/postgres.yml
---------------

#редактируем свои настройки из внутрянки пода 
-----------------------------------------------------------------------------
#команда на изменение параметра . ниже показан как я добавил раздел pg_hba::
patronictl -c postgres.yml edit-config
patronictl -c postgres.yml edit-config
patronictl -c postgres.yml edit-config

------------------
loop_wait: 10
maximum_lag_on_failover: 33554432
postgresql:
  parameters:
    archive_mode: 'on'
    archive_timeout: 1800s
    autovacuum_analyze_scale_factor: 0.02
    autovacuum_max_workers: 5
    autovacuum_vacuum_scale_factor: 0.05
    checkpoint_completion_target: 0.9
    hot_standby: 'on'
    log_autovacuum_min_duration: 0
    log_checkpoints: 'on'
    log_connections: 'on'
    log_disconnections: 'on'
    log_line_prefix: '%t [%p]: [%l-1] %c %x %d %u %a %h '
    log_lock_waits: 'on'
    log_min_duration_statement: 500
    log_statement: ddl
    log_temp_files: 0
    max_connections: 126
    max_replication_slots: 10
    max_wal_senders: 10
    tcp_keepalives_idle: 900
    tcp_keepalives_interval: 100
    track_functions: all
    wal_compression: 'on'
    wal_level: hot_standby
    wal_log_hints: 'on'
  pg_hba:
  - host all all 10.129.0.24/32 md5
  - host all all 10.128.0.6/32 md5
  - host all all 10.129.0.19/32 md5
  - host all all 10.130.0.34/32 md5
  use_pg_rewind: true
  use_slots: true
retry_timeout: 10
ttl: 30
----------------------

#подключаемся сначало локально меняем пароль пользователя postgres
psql -U postgres -d postgres
psql -U postgres -d postgres

alter user poargres with password 'mypass';
alter user poargres with password 'mypass';

#а после уже можем подключаться к ip адресу нашего сервиса
psql -U postgres -h 10.105.5.80 -d postgres
psql -U postgres -h 10.105.5.80 -d postgres

#если требуется изменить конфигурацию ккластера zalandopatroni01
patronictl -c postgres.yml restart zalandopatroni01
patronictl -c postgres.yml restart zalandopatroni01
patronictl -c postgres.yml restart zalandopatroni01

patronictl -c postgres.yml reload zalandopatroni01
patronictl -c postgres.yml reload zalandopatroni01
patronictl -c postgres.yml reload zalandopatroni01

patronictl -c postgres.yml restart zalandopatroni01
patronictl -c postgres.yml restart zalandopatroni01
patronictl -c postgres.yml restart zalandopatroni01
-----------------------------------------------------------------------------
#показать лидера и кворум из внутрянки пода 
-----------------------------------------------------------------------------
показать лидера и кворум из внутрянки пода 
patronictl -c postgres.yml list
patronictl -c postgres.yml list
patronictl -c postgres.yml list
-----------------------------------------------------------------------------

#подключаемся для тестов с управляюшей ноды
psql -U postgres -h 10.105.5.80 -d postgres
psql -U postgres -h 10.105.5.80 -d postgres
psql -U postgres -h 10.105.5.80 -d postgres

#БАЗОВАЯ ИНФОРМАЦИЯ порт версия сервер
select
inet_server_addr( ) AS "Server",
inet_server_port( ) AS "Port",
current_database() AS "CurrentDatabase",
version() AS "Version";

--------------------------
  Server   | Port | CurrentDatabase |                                                              Version
------------+------+-----------------+-----------------------------------------------------------------------------------------------------------------------------------
 10.244.2.4 | 5432 | postgres        | PostgreSQL 15.2 (Ubuntu 15.2-1.pgdg22.04+1) on x86_64-pc-linux-gnu, compiled by gcc (Ubuntu 11.3.0-1ubuntu1~22.04) 11.3.0, 64-bit
(1 row)
--------------------------

#показать ноды реплики
select usename,application_name,client_addr,backend_start,state,sync_state from pg_stat_replication;

--------------------------
postgres=# select usename,application_name,client_addr,backend_start,state,sync_state from pg_stat_replication;
 usename |  application_name  | client_addr |         backend_start         |   state   | sync_state
---------+--------------------+-------------+-------------------------------+-----------+------------
 standby | zalandopatroni01-0 | 10.244.1.4  | 2023-08-15 07:33:41.148787+00 | streaming | async
 standby | zalandopatroni01-2 | 10.244.3.6  | 2023-08-15 07:33:41.712539+00 | streaming | async
(2 rows)
--------------------------

#показать состояние конкретной ноды к которой подключился не находится ли она в состоянии восстоновления
select pg_is_in_recovery();

#выполняю преключение
kubectl -n spilo exec -it pod/zalandopatroni01-1 -- patronictl list

+ Cluster: zalandopatroni01 ------+---------+---------+----+-----------+
| Member             | Host       | Role    | State   | TL | Lag in MB |
+--------------------+------------+---------+---------+----+-----------+
| zalandopatroni01-0 | 10.244.1.4 | Replica | running |  1 |         0 |
| zalandopatroni01-1 | 10.244.2.4 | Leader  | running |  1 |           |
| zalandopatroni01-2 | 10.244.3.6 | Replica | running |  1 |         0 |
+--------------------+------------+---------+---------+----+-----------+

#выполняю switchover
kubectl -n spilo exec -it pod/zalandopatroni01-1 -- patronictl switchover
kubectl -n spilo exec -it pod/zalandopatroni01-1 -- patronictl switchover


#видим лидер изменился и TL
kubectl -n spilo exec -it pod/zalandopatroni01-1 -- patronictl list
+ Cluster: zalandopatroni01 ------+---------+---------+----+-----------+
| Member             | Host       | Role    | State   | TL | Lag in MB |
+--------------------+------------+---------+---------+----+-----------+
| zalandopatroni01-0 | 10.244.1.4 | Leader  | running |  2 |           |
| zalandopatroni01-1 | 10.244.2.4 | Replica | running |  2 |         0 |
| zalandopatroni01-2 | 10.244.3.6 | Replica | running |  2 |         0 |
+--------------------+------------+---------+---------+----+-----------+


#преподключаемся к ip сервиса и видем что мы уже на новом мастере
kubectl get all -A
psql -U postgres -h 10.105.5.80 -d postgres
select pg_is_in_recovery();

---------------------
postgres=# select pg_is_in_recovery();
 pg_is_in_recovery
-------------------
 f
(1 row)
---------------------


#создаю аварийную ситуацию роняю под лидера на котором работает кластер патрони прегружаю vm kub1

[root@masterkub vorori]# kubectl -n spilo exec -it pod/zalandopatroni01-1 -- patronictl list
+ Cluster: zalandopatroni01 ------+---------+---------+----+-----------+
| Member             | Host       | Role    | State   | TL | Lag in MB |
+--------------------+------------+---------+---------+----+-----------+
| zalandopatroni01-0 | 10.244.1.4 | Leader  | running |  2 |           |
| zalandopatroni01-1 | 10.244.2.4 | Replica | running |  2 |         0 |
| zalandopatroni01-2 | 10.244.3.6 | Replica | running |  2 |         0 |
+--------------------+------------+---------+---------+----+-----------+

shutdown -r now


#смотрим что под zalandopatroni01-2 стал лидером а zalandopatroni01-0 Replica и с лагом репликации 1 и TL не преключился на 3
root@kub3 vorori]# kubectl -n spilo exec -it pod/zalandopatroni01-1 -- patronictl list
+ Cluster: zalandopatroni01 ------+---------+---------+----+-----------+
| Member             | Host       | Role    | State   | TL | Lag in MB |
+--------------------+------------+---------+---------+----+-----------+
| zalandopatroni01-0 | 10.244.1.4 | Replica | running |  2 |         1 |
| zalandopatroni01-1 | 10.244.2.4 | Replica | running |  3 |         0 |
| zalandopatroni01-2 | 10.244.3.6 | Leader  | running |  3 |           |
+--------------------+------------+---------+---------+----+-----------+

#делаем принудительную синхронизацию с мастерос для пода zalandopatroni01-0
# можно выполнить специальную команду которая пресоздаст нашу реплику с нуля
# мы же попробуем просто убить под с этой репликой и посмотрим на работу kubernets


### единоразово смотрим логи pod видм кто у нас матер
kubectl logs --namespace spilo pod/zalandopatroni01-0 
kubectl logs --namespace spilo pod/zalandopatroni01-0 --tail=40 
kubectl logs --namespace spilo pod/zalandopatroni01-0 --tail=40 

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
psycopg2.OperationalError: connection to server at "10.244.3.6", port 5432 failed: FATAL:  password authentication failed for user "postgres"
connection to server at "10.244.3.6", port 5432 failed: FATAL:  no pg_hba.conf entry for host "10.244.1.5", user "postgres", database "postgres", no encryption
-------------------


kubectl get pods --namespace spilo
kubectl get pods --namespace spilo
kubectl get pods --namespace spilo

NAME                 READY   STATUS    RESTARTS      AGE
zalandopatroni01-0   1/1     Running   1 (18m ago)   3h24m
zalandopatroni01-1   1/1     Running   0             3h24m
zalandopatroni01-2   1/1     Running   0             3h24m

#удаляем проблемный под
kubectl delete pod zalandopatroni01-0 --namespace spilo
kubectl delete pod zalandopatroni01-0 --namespace spilo
kubectl delete pod zalandopatroni01-0 --namespace spilo

#проверяем под стартанул
kubectl get pods --namespace spilo
NAME                 READY   STATUS    RESTARTS   AGE
zalandopatroni01-0   1/1     Running   0          95s
zalandopatroni01-1   1/1     Running   0          3h28m
zalandopatroni01-2   1/1     Running   0          3h28m


### смотреть логи в режиме реального времени pod pod/zalandopatroni01-0 namespace spilo используя префикс -f
kubectl logs --namespace spilo pod/zalandopatroni01-0 -f
kubectl logs --namespace spilo pod/zalandopatroni01-0 -f


###умышленнос сломал zalandopatroni01-0
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

#наблюдаем вот такую картину
kubectl -n spilo exec -it pod/zalandopatroni01-1 -- patronictl list
+ Cluster: zalandopatroni01 ------+---------+--------------+----+-----------+
| Member             | Host       | Role    | State        | TL | Lag in MB |
+--------------------+------------+---------+--------------+----+-----------+
| zalandopatroni01-0 | 10.244.1.7 | Replica | start failed |    |   unknown |
| zalandopatroni01-1 | 10.244.2.4 | Replica | running      |  3 |         0 |
| zalandopatroni01-2 | 10.244.3.6 | Leader  | running      |  3 |           |
+--------------------+------------+---------+--------------+----+-----------+


#чтобы вернуть оживить под pod/zalandopatroni01-1  я пошел на сервер kub1 и почистил каталог с данными
#после чего патрони пресоздал реплику pod/zalandopatroni01-1
#после чего мы видим что с кластером все ок
kubectl -n spilo exec -it pod/zalandopatroni01-1 -- patronictl list
+ Cluster: zalandopatroni01 ------+---------+---------+----+-----------+
| Member             | Host       | Role    | State   | TL | Lag in MB |
+--------------------+------------+---------+---------+----+-----------+
| zalandopatroni01-0 | 10.244.1.7 | Replica | running |  3 |         0 |
| zalandopatroni01-1 | 10.244.2.4 | Replica | running |  3 |         0 |
| zalandopatroni01-2 | 10.244.3.6 | Leader  | running |  3 |           |
+--------------------+------------+---------+---------+----+-----------+

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
