#### 0) на почитать

Настройка кластера Kubernetes Centos 7
https://www.youtube.com/watch?v=5JBLITD3u6I
https://www.youtube.com/watch?v=5JBLITD3u6I
https://www.youtube.com/watch?v=5JBLITD3u6I

https://www.youtube.com/watch?v=XfWE7NqgbLE&list=PLmxqUDFl0XM6pr2y6tK51cHOFajhFsWG8&index=4
https://www.youtube.com/watch?v=XfWE7NqgbLE&list=PLmxqUDFl0XM6pr2y6tK51cHOFajhFsWG8&index=4
https://www.youtube.com/watch?v=XfWE7NqgbLE&list=PLmxqUDFl0XM6pr2y6tK51cHOFajhFsWG8&index=4 

Kubernetes 1.26 — электрифицирующая установка релиза
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

#### 0)

hostname masterkub ip  
hostname kub1      ip  
hostname kub2      ip  
hostname kub3      ip  

tail -f /var/log/messages
tail -f /var/log/messages
tail -f /var/log/messages


==========================================================================================================================================================================
hostname && hostname -i
hostname && hostname -i
hostname && hostname -i

1) добавляю на каждой ноде 
sudo cat <<EOF>> /etc/hosts
10.129.0.36 masterkub.ru-central1.internal
10.129.0.34 node1 kub1.ru-central1.internal
10.129.0.28 node2 kub2.ru-central1.internal
10.129.0.5 node3 kub3.ru-central1.internal
EOF

cat /etc/hosts
vim /etc/hosts

Шаг 1: Предварительные требования 
1.a.. Проверьте ОС, конфигурацию оборудования и Сетевое подключение 
1.b.. Отключите подкачку Disable SWAP и брандмауэр 

sudo yum -y install epel-release && yum install -y htop mc vim wget telnet git
sudo yum -y install epel-release && yum install -y htop mc vim wget telnet git
sudo yum -y install epel-release && yum install -y htop mc vim wget telnet git

Прежде чем приступить, нужно проверить сводную информацию об использовании и доступности подкачки на устройстве хранения. 
С помощью команды swapon: (Если команда ничего не возвращает, значит файла подкачки не существует)
swapon -s

sudo sed -i '/swap/d' /etc/fstab && sudo swapoff -a 
cat /etc/fstab
sudo systemctl stop firewalld 
sudo systemctl disable firewalld 

1.c Отключить Selinux 
Контейнеры должны получить доступ к файловой системе хоста. SELinux должен быть
установлен в разрешающий режим, который эффективно отключает его функции безопасности.

sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config && sudo setenforce 0

Шаг 2. Настройте локальные таблицы IP для просмотра мостового трафика 
2.a.. Включите мостовой трафик 

sudo modprobe br_netfilter && lsmod | grep br_netfilter
sudo modprobe br_netfilter && lsmod | grep br_netfilter

2.b.. Скопируйте приведенное ниже содержимое в этот файл.. /etc/modules-load.d/k8s.conf
cat <<EOF | > sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat /etc/modules-load.d/k8s.conf

копирую приведенное ниже содержимое в этот файл.. /etc/sysctl.d/k8s.conf 
cat <<EOF | > sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

cat /etc/sysctl.d/k8s.conf 

sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

cat /etc/sysctl.d/kubernetes.conf

проверяю , что вы загружаете модули
sudo modprobe overlay && sudo modprobe br_netfilter && sudo sysctl --system
sudo modprobe overlay && sudo modprobe br_netfilter && sudo sysctl --system
sudo modprobe overlay && sudo modprobe br_netfilter && sudo sysctl --system

Шаг 3. Установите Docker будем работать через его движок

3.a.. удаляю все старые версии если чтото было установлено
sudo yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine
sudo yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine
sudo yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine

3. б.. устанавливаю утилиты Yum | Диспетчер конфигурации  
sudo yum install -y yum-utils
sudo yum install -y yum-utils
sudo yum install -y yum-utils

3.c.. настраиваю репозиторий Docker
yum-config-manager  --add-repo https://download.docker.com/linux/centos/docker-ce.repo

3. d.. устанавливаю Docker Engine, Docker CLI, Docker RUNTIME $ 
yum install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
yum install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

4.b. копирую е приведенное ниже содержимое в этот файл.. /etc/docker/
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

cat /etc/docker/daemon.json

sudo systemctl enable docker && sudo systemctl restart docker && sudo systemctl status docker
sudo systemctl daemon-reload

Шаг 5. устанавливаем kubeadm, kubectl, kubelet 
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

cat /etc/yum.repos.d/kubernetes.repo

reboot -h now
reboot -h now
reboot -h now

смотрим какие версии доступны для установки
yum --showduplicates list kubeadm.x86_64
yum --showduplicates list kubeadm.x86_64
yum --showduplicates list kubeadm.x86_64

релизы кубера
https://kubernetes.io/releases/
https://kubernetes.io/releases/
https://kubernetes.io/releases/

релизы citus
https://github.com/citusdata/docker/blob/master/CHANGELOG.md
https://github.com/citusdata/docker/blob/master/CHANGELOG.md
https://github.com/citusdata/docker/blob/master/CHANGELOG.md

Устанавливаем командой !
yum install -y kubelet-1.26.1-0.x86_64 kubeadm-1.26.1-0.x86_64 kubectl-1.26.1-0.x86_64
yum install -y kubelet-1.26.1-0.x86_64 kubeadm-1.26.1-0.x86_64 kubectl-1.26.1-0.x86_64

yum install -y kubelet-1.24.15-0.x86_64 kubeadm-1.24.15-0.x86_64 kubectl-1.24.15-0.x86_64
yum remove -y kubelet-1.24.15-0.x86_64 kubeadm-1.24.15-0.x86_64 kubectl-1.24.15-0.x86_64

yum install -y kubelet-1.21.0-0.x86_64 kubeadm-1.21.0-0.x86_64 kubectl-1.21.0-0.x86_64
yum install -y kubelet-1.21.0-0.x86_64 kubeadm-1.21.0-0.x86_64 kubectl-1.21.0-0.x86_64

Устанавливаем командой old
yum install -y kubelet-1.16.2-0.x86_64 kubeadm-1.16.2-0.x86_64 kubectl-1.16.2-0.x86_64

добавляем в автозагрузку
sudo systemctl enable --now kubelet && systemctl status kubelet
sudo systemctl enable --now kubelet && systemctl status kubelet
sudo systemctl enable --now kubelet && systemctl status kubelet

tail -f /var/log/messages
tail -f /var/log/messages
tail -f /var/log/messages

----------------------------------------------------------------------------------------------------------------------------
ошибка:
Jul 13 06:55:55 masterkub kubelet: E0713 06:55:55.831658    1708 run.go:74] "command failed" err="failed to validate kubelet flags: 
the container runtime endpoint address was not specified or empty, use --container-runtime-endpoint to set"

systemctl restart containerd && sudo systemctl enable --now containerd && systemctl status containerd
systemctl restart containerd && sudo systemctl enable --now containerd && systemctl status containerd
systemctl restart containerd && sudo systemctl enable --now containerd && systemctl status containerd

systemctl status containerd
systemctl status containerd
systemctl status containerd

смотрим интерфейсы
ip -4 addr show
ip -4 addr show
ip -4 addr show

rm /etc/containerd/config.toml
rm /etc/containerd/config.toml
rm /etc/containerd/config.toml

systemctl restart containerd && sudo systemctl enable --now containerd && systemctl status containerd
systemctl restart containerd && sudo systemctl enable --now containerd && systemctl status containerd
systemctl restart containerd && sudo systemctl enable --now containerd && systemctl status containerd
----------------------------------------------------------------------------------------------------------------------------



----------------------------------------------------------------------------------------------------------------------------
Pull the images , извлекает образы для версии Kubernetes 1.26.
sudo kubeadm config images pull --image-repository=registry.k8s.io --cri-socket unix:///run/containerd/containerd.sock --kubernetes-version v1.26.1
sudo kubeadm config images pull --image-repository=registry.k8s.io --cri-socket unix:///run/containerd/containerd.sock --kubernetes-version v1.26.1
sudo kubeadm config images pull --image-repository=registry.k8s.io --cri-socket unix:///run/containerd/containerd.sock --kubernetes-version v1.26.1
----------------------------------------------------------------------------------------------------------------------------

----------------------------------------------------------------------------------------------------------------------------
запускаю команду инициализации kubeadm на узле плоскости управления.
Здесь CIDR сети pod зависит от CNI, который вы будете устанавливать позже, поэтому в этом случае я использую фланель 
и --control-plane-endpoint буду общедоступным IP-адресом для экземпляра (это также может быть частный IP-адрес, 
но если вы хотите получить доступ это из-за пределов узла с помощью Kubeconfig, тогда вам нужно указать общедоступный IP-адрес).

sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=10.129.0.36 --upload-certs --kubernetes-version=v1.26.1 --control-plane-endpoint=masterkub.ru-central1.internal --cri-socket unix:///run/containerd/containerd.sock
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=10.129.0.36 --upload-certs --kubernetes-version=v1.26.1 --control-plane-endpoint=masterkub.ru-central1.internal --cri-socket unix:///run/containerd/containerd.sock
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=10.129.0.36 --upload-certs --kubernetes-version=v1.26.1 --control-plane-endpoint=masterkub.ru-central1.internal --cri-socket unix:///run/containerd/containerd.sock
----------------------------------------------------------------------------------------------------------------------------


----------------------------------------------------------------------------------------------------------------------------
kubectl version
kubectl version
kubectl version

WARNING: This version information is deprecated and will be replaced with the output from kubectl version --short.  Use --output=yaml|json to get the full version.
Client Version: version.Info{Major:"1", Minor:"26", GitVersion:"v1.26.1", GitCommit:"8f94681cd294aa8cfd3407b8191f6c70214973a4", GitTreeState:"clean", BuildDate:"2023-01-18T15:58:16Z", GoVersion:"go1.19.5", Compiler:"gc", Platform:"linux/amd64"}
Kustomize Version: v4.5.7
Server Version: version.Info{Major:"1", Minor:"26", GitVersion:"v1.26.1", GitCommit:"8f94681cd294aa8cfd3407b8191f6c70214973a4", GitTreeState:"clean", BuildDate:"2023-01-18T15:51:25Z", GoVersion:"go1.19.5", Compiler:"gc", Platform:"linux/amd64"}
----------------------------------------------------------------------------------------------------------------------------

----------------------------------------------------------------------------------------------------------------------------
hostname -i
hostname -i
hostname -i

10.129.0.36
----------------------------------------------------------------------------------------------------------------------------

выполняю
----------------------------------------------------------------------------------------------------------------------------
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=10.129.0.36 --upload-certs --kubernetes-version=v1.26.1 --control-plane-endpoint=masterkub.ru-central1.internal --cri-socket unix:///run/containerd/containerd.sock
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=10.129.0.36 --upload-certs --kubernetes-version=v1.26.1 --control-plane-endpoint=masterkub.ru-central1.internal --cri-socket unix:///run/containerd/containerd.sock
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=10.129.0.36 --upload-certs --kubernetes-version=v1.26.1 --control-plane-endpoint=masterkub.ru-central1.internal --cri-socket unix:///run/containerd/containerd.sock
[root@masterkub ~]# sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=10.129.0.36 --upload-certs --kubernetes-version=v1.26.1 --control-plane-endpoint=masterkub.ru-central1.internal --cri-socket unix:///run/containerd/containerd.sock
[init] Using Kubernetes version: v1.26.1
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local masterkub.ru-central1.internal] and IPs [10.96.0.1 10.129.0.36]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [localhost masterkub.ru-central1.internal] and IPs [10.129.0.36 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [localhost masterkub.ru-central1.internal] and IPs [10.129.0.36 127.0.0.1 ::1]
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
[apiclient] All control plane components are healthy after 7.002298 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Storing the certificates in Secret "kubeadm-certs" in the "kube-system" Namespace
[upload-certs] Using certificate key:
2e3e131173d1ac126ea18d8c0ef049a3089152ae6875570f84ff3431fd9be74b
[mark-control-plane] Marking the node masterkub.ru-central1.internal as control-plane by adding the labels: [node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node masterkub.ru-central1.internal as control-plane by adding the taints [node-role.kubernetes.io/control-plane:NoSchedule]
[bootstrap-token] Using token: 6ar5s6.jvgjuszqhzdlmj33
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

  kubeadm join masterkub.ru-central1.internal:6443 --token 6ar5s6.jvgjuszqhzdlmj33 \
        --discovery-token-ca-cert-hash sha256:bbc69ceafaa3261fc3d7ce088f9f0837318d2e1dbc5f426b7de264f9c5bf64db \
        --control-plane --certificate-key 2e3e131173d1ac126ea18d8c0ef049a3089152ae6875570f84ff3431fd9be74b

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join masterkub.ru-central1.internal:6443 --token 6ar5s6.jvgjuszqhzdlmj33 \
        --discovery-token-ca-cert-hash sha256:bbc69ceafaa3261fc3d7ce088f9f0837318d2e1dbc5f426b7de264f9c5bf64db
----------------------------------------------------------------------------------------------------------------------------



----------------------------------------------------------------------------------------------------------------------------
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
export KUBECONFIG=/etc/kubernetes/admin.conf
kubectl apply -f https://github.com/coreos/flannel/raw/master/Documentation/kube-flannel.yml
----------------------------------------------------------------------------------------------------------------------------


----------------------------------------------------------------------------------------------------------------------------
https://stackoverflow.com/questions/52098214/kube-flannel-in-crashloopbackoff-status
https://stackoverflow.com/questions/52098214/kube-flannel-in-crashloopbackoff-status
https://stackoverflow.com/questions/52098214/kube-flannel-in-crashloopbackoff-status

ошибка:
kube-system   kube-flannel-ds-amd64-42rl7            0/1       CrashLoopBackOff
как оказалась проблема была в этом --pod-network-cidr=10.244.0.0/16
Для flannelкорректной работы необходимо перейти --pod-network-cidr=10.244.0.0/16на kubeadm init.
Это верно только для главного узла. Другие рабочие узлы не должны запускать эту команду. 
Посмотрите на другой ответ, предоставленный @pande ниже; это то, что решило эту проблему для меня.


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



вспомогательные команды
----------------------------------------------------------
kubectl get pods --all-namespaces -o wide
kubectl get pods --all-namespaces -o wide
kubectl get pods --all-namespaces -o wide

#Это предоставит информацию обо всех deployments во всех namespaces
kubectl get deployments --all-namespaces
kubectl get deployments --all-namespaces
kubectl get deployments --all-namespaces

#Это предоставит информацию обо всех deployments в namespaces cert-manager
kubectl get deployments -n cert-manager
kubectl get deployments -n cert-manager
kubectl get deployments -n cert-manager


#Это предоставит информацию обо всех модулях, развертываниях, службах и заданиях в пространстве имен.
kubectl get pods,services,deployments,jobs
kubectl get pods,services,deployments,jobs
kubectl get pods,services,deployments,jobs

#Это предоставит информацию о сервисахесли
kubectl get services
kubectl get services
kubectl get services

#Это предоставит информацию о пространствах имен
kubectl get namespace
kubectl get namespace
kubectl get namespace

#Чтобы перечислить все развертывания:
kubectl get deployments --all-namespaces
kubectl get deployments --all-namespaces
kubectl get deployments --all-namespaces

kubectl get all
kubectl get all
kubectl get all

#показать все поды даже системные
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

kubectl get pv
kubectl get all -o wide

-- посмотрим дефолтный тип стораджа
kubectl get storageclasses

kubectl get pvc -o wide
kubectl get pv -o wide


#прозвонить pod
kubectl describe pod -n kube-system etcd-masterkub.ru-central1.internal
kubectl describe pod -n kube-system etcd-masterkub.ru-central1.internal
kubectl describe pod -n kube-system etcd-masterkub.ru-central1.internal

#логи pod
kubectl logs --namespace kube-system etcd-masterkub.ru-central1.internal
kubectl logs --namespace kube-system etcd-masterkub.ru-central1.internal
kubectl logs --namespace kube-system etcd-masterkub.ru-central1.internal
kubectl logs --namespace kube-system kube-flannel-ds-amd64-5fx2p
kubectl logs --namespace kube-system kube-flannel-ds-amd64-5fx2p
kubectl logs --namespace kube-system kube-flannel-ds-amd64-5fx2p

---
смотрим интерфейсы
ip -4 addr show
ip -4 addr show
ip -4 addr show

Удалите интерфейс cni0и файлы flannel.1.
sudo ip link delete cni0;
sudo ip link delete flannel.1;
---

tail -f /var/log/messages
tail -f /var/log/messages
tail -f /var/log/messages
----------------------------------------------------------


----------------------------------------------------------------------------------------------------------------------------

смотрим что все работает
----------------------------------------------------------------------------------------------------------------------------
root@masterkub vorori]# kubectl get nodes
NAME                             STATUS   ROLES           AGE     VERSION
masterkub.ru-central1.internal   Ready    control-plane   4m27s   v1.26.1
----------------------------------------------------------------------------------------------------------------------------

смотрим что все работает
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




пытаемся присоединиться к мастеру
----------------------------------------------------------------------------------------------------------------------------
rm /etc/containerd/config.toml
rm /etc/containerd/config.toml
rm /etc/containerd/config.toml

systemctl restart containerd && sudo systemctl enable --now containerd && systemctl status containerd
systemctl restart containerd && sudo systemctl enable --now containerd && systemctl status containerd
systemctl restart containerd && sudo systemctl enable --now containerd && systemctl status containerd


kubeadm join masterkub.ru-central1.internal:6443 --token 6ar5s6.jvgjuszqhzdlmj33 --discovery-token-ca-cert-hash sha256:bbc69ceafaa3261fc3d7ce088f9f0837318d2e1dbc5f426b7de264f9c5bf64db


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



---------------------------------

можно скопировать файл kubeconfig с узла controlplane node управления (~/.kube/config ) 
на локальный и экспортировать переменную KUBECONFIG или получить прямой доступ к кластеру с узла controlplane node.

cat /etc/kubernetes/admin.conf
cat /etc/kubernetes/admin.conf
cat /etc/kubernetes/admin.conf

apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUMvakNDQWVhZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJek1EY3hNekE1TXpnMU9Gb1hEVE16TURjeE1EQTVNemcxT0Zvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTmhaCk1rRnVWenEyOHVRcWZtRnphR2dCQWt3R1BNNEFJSkdudnZYSHBDVlNKM2tZU0dseXNuM1hjUExRTXlRRUpwVnIKbU5GOXMxdDI3V1BjQVBDZVM5RkNsZEFZamVJaENReEtUcXRGZjZwSjJoY2ZocGpIVjJaenBsTWlxNTBqU1FBagozcGZzYXlNWnJBODBxVmtGcS9Nb3JyNjloTkhHNE9NM0dMVmJiM2o1U0NHTDQvMFFHcEZkWmVPMXhtOTByVzZOCi9OY2l5ZlFrcVlHdXBBaEhqcGRCUkJvSnBJdXdmYUpwMDFYYUVxbkUvVkZ4SU1ETzNQUkVXR0xqRjRtT3NoNHgKc2YvNFRaMzRYb0Jta1V0cjh1cTh2WDhjM3ZGNXgzY045OXlmRlRWbFcrV2kxbWo0SUdKeXBzZ3NGNUcwUjVtdwpCWGRtcFkyVFdNVjNvSDlPa1Y4Q0F3RUFBYU5aTUZjd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZCS2RMcEgzWGsrYmhLdFVlWDZQSC9zcWtrck5NQlVHQTFVZEVRUU8KTUF5Q0NtdDFZbVZ5Ym1WMFpYTXdEUVlKS29aSWh2Y05BUUVMQlFBRGdnRUJBRzVTYU9nQWNNV1lJTmNka2JlZQpRWjVQVzY3MVBRTWxFTTNOYUIrdEQ1REY3VmEyRTRNRTRlb2NkSy83TGZxa3prUGlsM0NvZWtTRVJvZW5CU3NCCk9reW1XaVpWdVJBWGs2QVdVdEMrRHVkNEpib2cwSXNFRCswYlZuZ1paZEsvQUl1Z1pEL01CTGMydTZnZVprb2QKU28xUXYzaGd3MVUrbzZNK2FBVWk3K2U0RjBESXR0VTNUNXgyZ3BwMUdkU1pIRzVtOXh6ZGpIeVpKM1hRb2R2Rwo5d0I0MG9GMGEzSkdYandSekRqU0s0Ri9oUmVsUndxenVaaEpVNXYycjBUVmdyVUszcUpnNDhGSFkzWmVtOWVUCjlKRGJteW1XYkorRmdHQi90NjBJUU1oS2pqOEZucXIyT1hsdmJuaVZjUXp1YVBTcENSOHVGM3ZXcEdOZU5Za1MKSjkwPQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
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
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURJVENDQWdtZ0F3SUJBZ0lJTktFK3RRT1NLeFF3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TXpBM01UTXdPVE00TlRoYUZ3MHlOREEzTVRJd09UTTVNREZhTURReApGekFWQmdOVkJBb1REbk41YzNSbGJUcHRZWE4wWlhKek1Sa3dGd1lEVlFRREV4QnJkV0psY201bGRHVnpMV0ZrCmJXbHVNSUlCSWpBTkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQXNaWVJZZE5CUHJZdXhDOTMKWk92eEJSSXU0S0F0U0g2bVlPelNZcTMzbEJCWkFJdUNpRjlGWnczMnZpSi9vbXhEZVlhMWw2czVsYndqNDNvUwpmSThxaS9mdDk3bUF3R2s5cThYQUVIUjRoMFo4QnRQR1VlSW11NUJtamJpS2VCQy9CQnpXUnYydTFHVCtRZlcyCnFXd0FaRS9VNUIwK1Z5VHJFeEEwbFE0WnU4d0VLTEtmczhXeFM2Y2RYK3Rkbi9Qams1OGZmTElyUXp5ZHM1Vm8KaGV4eVcxMHNwRkNjSDVwQktDT3JpUXRPL3RRWTBudjl2VzBVSEgydXNhL2c4eUF4N3JsejlYUU5NMENYc3ptSQpvRW9yeDNGRjkzcnBQaGNMRmJqR2kxWmlVakF4TWllZTltRnBWQStJSGpGdVBFck05UU0vanZDVUozalh1WHJvCkkvd2RzUUlEQVFBQm8xWXdWREFPQmdOVkhROEJBZjhFQkFNQ0JhQXdFd1lEVlIwbEJBd3dDZ1lJS3dZQkJRVUgKQXdJd0RBWURWUjBUQVFIL0JBSXdBREFmQmdOVkhTTUVHREFXZ0JRU25TNlI5MTVQbTRTclZIbCtqeC83S3BKSwp6VEFOQmdrcWhraUc5dzBCQVFzRkFBT0NBUUVBWkI5MFFldmE0N2UrVUNYd2pmWkF3RzhwS1ZZdTBuODEzbko5CnFHamROZ0FJdm9SaGhoSFNXcERsUzVObGpWckorSWE4Zk50RC9HcTNxOG1kVmhmdDZLR2Mxa1FWUEFVMkNycXUKQjFYYUUraUxtSVZQUWRvS2R2MTR1WDQrTHQwQ0NORGNZWlNWd3hHT0xCZXcvakVXVitUa0ViSlJTbDlMOHlVSApsYitOOHNNOWtBOE5sdGhnNGFNcS8wb05Nek1xYm05bTQ3T3RwWXhGTkp0bGd5VDJxWXpWczJES1NkYm1ZdGVQCjZ3OS9PcHkwNGtqTUJHWTdxVnJudkY3OWhJeVlSUlNwb0tiR29ucFZPRkcxbTgrYmg3SEJuYmZtTGVQcmNnbWsKQ3hST2JVcEtRTThvNlJ2WjhxZlI2MGFMTWdVaTFjbVNVRllUYUFEclVpTHlsNkhobGc9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcEFJQkFBS0NBUUVBc1pZUllkTkJQcll1eEM5M1pPdnhCUkl1NEtBdFNINm1ZT3pTWXEzM2xCQlpBSXVDCmlGOUZadzMydmlKL29teERlWWExbDZzNWxid2o0M29TZkk4cWkvZnQ5N21Bd0drOXE4WEFFSFI0aDBaOEJ0UEcKVWVJbXU1Qm1qYmlLZUJDL0JCeldSdjJ1MUdUK1FmVzJxV3dBWkUvVTVCMCtWeVRyRXhBMGxRNFp1OHdFS0xLZgpzOFd4UzZjZFgrdGRuL1BqazU4ZmZMSXJRenlkczVWb2hleHlXMTBzcEZDY0g1cEJLQ09yaVF0Ty90UVkwbnY5CnZXMFVISDJ1c2EvZzh5QXg3cmx6OVhRTk0wQ1hzem1Jb0VvcngzRkY5M3JwUGhjTEZiakdpMVppVWpBeE1pZWUKOW1GcFZBK0lIakZ1UEVyTTlRTS9qdkNVSjNqWHVYcm9JL3dkc1FJREFRQUJBb0lCQUR6blJwYlNLTElsTU95LwppcWJsOUFiVk1aWi8ySWZnUjVjK0lQa095TFhTMVhOR1pVSDN5b21KVkhIb3l2VWRSQmJOaUVHM3kwdjY5aVB4Cm5aMXlUQmtzc2tRZkxuVE9vd1BpRWZpU3VUTkxqUVdhWTQxNWplY05vdDQveDkxZGdPVWNMckN5Rjk5bnMzdFQKbFkyTktLVSthN2kvL3IxTVFBS05zRHJlVnh6OFRDVFErVHZHQVdUUjVjSTRweWZsTkFyUkx5ZllRYyszL2Q4QQp6VWNUUmVQdWNFbkFoa2hrVkZ5N2ZyZjgvMFR2WlBGZXdDRkpVVVRzNCtWK2RSNWNEd3AzQlVwVjdNdEhMVm5oClA3WmZOYTBGTnlabDFXN3V2SS84N0VTM1NURmJZWE1OQXFDR1FlbnlEcGhrTHFObnV0VDBlMHZwemtTQ1c3SzgKSVQ3eC9uRUNnWUVBNlpzd3Iwd1hoaG5JcU9XaHhTZ0JuZTlabCtKNUVzalRRWkhXV2hPRGdyN1ExeTV2ZVdEcAo3Z1FqeEZ3ZVZSZ0hGcWwwY3VSQ3d3b21mM2R2M1ZCOVpWRjBkTENvWXdTRFJvZTFCSXRXTGVNb0NZT3dMRTFkCmozcFd4VVhhL3VCNTUrS2ppdjQzWWVVNzFYdnRBOU1jd2hFdnJhZFJ6c0thU3dUMm4wU3JDY3NDZ1lFQXdwd2UKbUhkY2p0am1JQTNmbTZiSEZRZDhsc01ZeG1FOStBeHdnaC9aZFJ4TE9meGN5TXl3LytRVXV1VzRGdjRCaEhQTAo0enZVeGFJbDRTZ1hLYmJDaWRjeXIxZHdQSUcwUGxkL3F5RitpYzJRZlZWd0h5aUlnd3czb2FzaGVqZDY3NGZCCkVEZE13Mmp6T0F6a0VXUlMxRTAyd0NTMEJjb0JOOXhLMVlhVE52TUNnWUFrQXhlTFBvaTExSTI1YzhUdmRzNWgKQVgvblNUTnU4T1NZVEJvbVFySFlXd0FvMi9DMVhucFJoZlBabG5YYW1seWxZclFmN2c1WXNOemtjNDRjS1FkYgpzaVhvd3o4Q0hMVDhEM21aWEwySlQyWmhxUnBraWZ5dFhLZTV1NTRhQXBMb2Eyall5WDNTS3B1QnVwdjhKZlJkCituWkdKL0FWbHF4Z2VrQm1weGhTNXdLQmdRQzNRZkNGdnBESWEzTjQ1OWVUYVYrN2E4dGs1Tjhsb0ZpTWhwcEQKenQ0bHE3a2pKNFB3Q3VENmRyc1dyRS9JUnZVQzExQTk4UStSOC9rYnAzYjRid09PYmJscTZEbm1vSFVzNTVSdgpnQ0Q2ZnpyNjYwT2o4N1ZwUWszNHpYKys2Uk81RCtzNzYvYzdaRTcwang4TlNaMitFZC9tM0NreDhtRm5TdWlsClhnNnZCUUtCZ1FDUHJGd095TGxobGc0cVZYbnY3eXRUVWVnS2NUQk9uSU93VVBEWXBBemJWaDIwWmlFbUhiZ3YKckY2YlhmK0hlcWZSRVpLWjl1RzM5ckZESVRFOC9IUDE2QWUvWlc0Z3M2VSt3dmU4WjdZc1BxZ2dadnE3RHRMOQoyQUgvQlVyS0pNTGhnVkl1TzluVzlTaDVhWERFWXhVeFhpTFcveVFIUjNzWmdEeVlYcmIwK1E9PQotLS0tLUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=


vim $HOME/.kube/config
vim $HOME/.kube/config
vim $HOME/.kube/config

sudo chown $(id -u):$(id -g) $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config


проверяем
kubectl get nodes
kubectl get nodes
kubectl get nodes

[root@kub1 vorori]# kubectl get nodes
NAME                             STATUS   ROLES           AGE   VERSION
kub1.ru-central1.internal        Ready    <none>          16h   v1.26.1
kub2.ru-central1.internal        Ready    <none>          16h   v1.26.1
kub3.ru-central1.internal        Ready    <none>          16h   v1.26.1
masterkub.ru-central1.internal   Ready    control-plane   20h   v1.26.1

----------------------------------------------------------------------------------------------------------------------------




проверяю на главном узле
----------------------------------------------------------------------------------------------------------------------------
[root@masterkub vorori]# kubectl get nodes
NAME                             STATUS   ROLES           AGE     VERSION
kub1.ru-central1.internal        Ready    <none>          35m     v1.26.1
kub2.ru-central1.internal        Ready    <none>          35s     v1.26.1
kub3.ru-central1.internal        Ready    <none>          16m     v1.26.1
masterkub.ru-central1.internal   Ready    control-plane   4h30m   v1.26.1
----------------------------------------------------------------------------------------------------------------------------


важно! выделил  ---- ноды значения/наименования которых мы ввидем из разных команд
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
NAMESPACE      NAME                                                         READY   STATUS    RESTARTS      AGE
---
kube-flannel   pod/kube-flannel-ds-7g8q2                                    1/1     Running   0             37m
kube-flannel   pod/kube-flannel-ds-bsh77                                    1/1     Running   0             18m
kube-flannel   pod/kube-flannel-ds-d82s6                                    1/1     Running   0             3m17s
----
kube-flannel   pod/kube-flannel-ds-rdp9t                                    1/1     Running   1 (63m ago)   4h30m
kube-system    pod/coredns-787d4945fb-f98p8                                 1/1     Running   1 (63m ago)   4h33m
kube-system    pod/coredns-787d4945fb-s78vs                                 1/1     Running   1 (63m ago)   4h33m
kube-system    pod/etcd-masterkub.ru-central1.internal                      1/1     Running   2 (63m ago)   4h33m
kube-system    pod/kube-apiserver-masterkub.ru-central1.internal            1/1     Running   2 (63m ago)   4h33m
kube-system    pod/kube-controller-manager-masterkub.ru-central1.internal   1/1     Running   1 (63m ago)   4h33m
kube-system    pod/kube-proxy-6knkk                                         1/1     Running   0             18m
kube-system    pod/kube-proxy-8vf7j                                         1/1     Running   0             3m17s
kube-system    pod/kube-proxy-9khtr                                         1/1     Running   1 (63m ago)   4h33m
kube-system    pod/kube-proxy-n7xcg                                         1/1     Running   0             37m
kube-system    pod/kube-scheduler-masterkub.ru-central1.internal            1/1     Running   2 (63m ago)   4h33m

NAMESPACE     NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
default       service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP                  4h33m
kube-system   service/kube-dns     ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   4h33m

NAMESPACE      NAME                             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-flannel   daemonset.apps/kube-flannel-ds   4         4         4       4            4           <none>                   4h30m
kube-system    daemonset.apps/kube-proxy        4         4         4       4            4           kubernetes.io/os=linux   4h33m

NAMESPACE     NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
kube-system   deployment.apps/coredns   2/2     2            2           4h33m

NAMESPACE     NAME                                 DESIRED   CURRENT   READY   AGE
kube-system   replicaset.apps/coredns-787d4945fb   2         2         2       4h33m
----------------------------------------------------------------------------------------------------------------------------



разворачиваем цитус  time  1.08
----------------------------------------------------------------------------------------------------------------------------
mkdir /tmp/citus

echo -n '2338484' | base64 
echo -n '2338484' | base64 
echo -n '2338484' | base64 

-- пароль
-- посмотреть 
echo 'MjMzODQ4NA==' | base64 -d
echo 'MjMzODQ4NA==' | base64 -d
echo 'MjMzODQ4NA==' | base64 -d

-------------
vim /tmp/citus/postgres-storage.yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: postgres-pv-claim
  labels:
    app: postgres
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
-------------


-------------
vim /tmp/citus/secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: citus-secrets
type: Opaque
data:
  password: MjMzODQ4NA==
-------------

-------------
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
-------------


my copy
-------------
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
        - name: storage-mycitus-master
          mountPath: /var/pgsql-volume
        livenessProbe:
          exec:
            command:
            - ./pg_healthcheck
          initialDelaySeconds: 60
      volumes:
        - name: storage
          persistentVolumeClaim:
            claimName: citus-master-pvc
-------------


пример!!!!!!!!!!!!
https://github.com/BigKAA/youtube/blob/d4a0099ef236546b4ac5cc900e9ac4c47fd6ae63/base/Crunchy%20PostgreSQL%20Operator/01-pv-pvc.yaml
https://www.youtube.com/watch?v=kuTG8YGhJtY&t=1585s   time 21:32
mkdir /var/pgsql-volume
name: storage-main -- имя pv мастер
name: storage-repl -- имя pv реплика
- ReadWriteMany  -- доступ чтение запись
storage: 8Gi -- сколко gb
  local:     -- локальная файловая система
    path: /var/pgsql-volume -- по этому пути которому мы сороздали
  nodeAffinity: -- говорит нам о том что приземлять pv на ноде у которй есть лейбл  key: db равно  values: - pgsql-main
  nodeAffinity: -- говорит нам о том что приземлять pv на ноде у которй есть лейбл  key: db равно  values: - pgsql-repl
  
  надо посмотреть видео посвешенное pv
-------------
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: storage-main
  labels:
    storage: basemain
spec:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 8Gi
  local:
    path: /var/pgsql-volume
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: db
          operator: In
          values:
          - pgsql-main
  persistentVolumeReclaimPolicy: Retain
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: storage-repl
  labels:
    storage: baserepl
spec:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 8Gi
  local:
    path: /var/pgsql-volume
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: db
          operator: In
          values:
          - pgsql-repl
  persistentVolumeReclaimPolicy: Retain
-------------







пример!!!!!!!!!!!!
-------------
vim /tmp/citus/master.yaml
apiVersion: v1
kind: Service
metadata:
  name: postgres
  labels:
    app: postgres
spec:
  type: NodePort
  ports:
   - port: 5432
  selector:
    app: postgres

---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres-statefulset
spec:
  serviceName: "postgres"
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:latest
        ports:
        - containerPort: 5432
          name: postgredb
        env:
          - name: POSTGRES_DB
            value: myapp
          - name: POSTGRES_USER
            value: myuser
          - name: POSTGRES_PASSWORD
            value: passwd
        volumeMounts:
        - name: postgredb
          mountPath: /var/lib/postgresql/data
          subPath: postgres
  volumeClaimTemplates:
  - metadata:
      name: postgredb
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: standard
      resources:
        requests:
          storage: 1Gi

-------------



-------------
vim /tmp/citus/workers.yaml
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
        image: citusdata/citus:10.1-pg12
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
-------------


#выполняем
kubectl apply -f /tmp/citus/mypv.yaml
kubectl apply -f /tmp/citus/postgres-storage.yaml
kubectl get pvc -o wide
kubectl get pv -o wide


kubectl create -f /tmp/citus/secrets.yaml
kubectl apply -f /tmp/citus/master.yaml
kubectl apply -f /tmp/citus/workers.yaml

kubectl get all
kubectl get pods


----------------------------------------------------------------------------------------------------------------------------























----------------------------------------------------------------------------------------------------------------------------

-- настроим Citus руками в кубере
-- вспоминаем, что с нодами кластера
kubectl cluster-info
kubectl get nodes

!!!!!
-- Citus в кубере нет, но один китаец таки смог:
-- https://www.google.com/search?client=firefox-b-d&q=citus+kubernetes
-- https://github.com/aeuge/citus-k8s
-- образ старый 7.3.0, но не бесполезный
-- посмотрим скрипты
cd citus
nano secrets.yaml
nano master.yaml
nano workers.yaml
!!!!!


time 1.10
!!! не забываем указывать -n для переноса строки - посмотрим разницу
!!! echo 'otus321$' | base64 -- худший!!!!!
!!! echo 'otus321$' | base64 -- худший!!!!!
!!! echo 'otus321$' | base64 -- худший!!!!!


!!! echo -n 'otus321$' | base64   --- вот лучший способ при кодировании!!!!!!!!!!
!!! echo -n 'otus321$' | base64   --- вот лучший способ при кодировании!!!!!!!!!!
!!! echo -n 'otus321$' | base64   --- вот лучший способ при кодировании!!!!!!!!!!


kubectl create -f secrets.yaml
kubectl create -f master.yaml
-- чуть позже запускаем, после создания мастера
-- есть вероятность, что kubectl apply -f . не отработает
kubectl get all
kubectl create -f workers.yaml

-- обратите внимание, что мастер имеет случайное название, на воркеры строго определенное
kubectl get all

kubectl exec -it pod/citus-master-5bff7bc99c-rxl5b -- bash
psql -U postgres
SELECT * FROM master_get_active_worker_nodes(); 
create database test;

-- Заапргейдим наш цитус, добавив лоад балансер и еще 1 ноду
kubectl delete -f .
kubectl get pvc
kubectl get secrets
gcloud compute disks list
cd ../citus_LB
kubectl create -f secrets.yaml
nano master.yaml
kubectl create -f master.yaml

kubectl apply -f workers.yaml

kubectl get all

-- посмотреть секреты
kubectl get secret citus-secrets -o yaml

-- пароль otus321$
-- посмотреть 
echo 'b3R1czMyMSQ=' | base64 -d


kubectl get service
psql -h 34.31.239.206 -U postgres --password -p 5432
-- почему нет БД test?

SELECT * FROM master_get_active_worker_nodes();


-- добавим еще 1 ноду, для этого просто отредактируем стейтфул сет
nano workers.yaml
kubectl apply -f workers.yaml


kubectl get all
psql -h 35.192.66.109 -U postgres --password -p 5432
SELECT * FROM master_get_active_worker_nodes();

-- удалим все
kubectl delete all,ing,secrets,pvc,pv --all


-- посмотрим на сборки Цитуса
-- https://hub.docker.com/r/citusdata/citus/

/* описание проблем

-- если мы просто удалим наш деплоймент kubectl delete -f . , pvc все равно остануться

-- после попыток развернуть 9.4.0 вернулся к 7.3.0
-- kubectl logs pod/citus-master-68959cc849-btn9l
-- 2020-08-18 10:26:24.838 UTC [1] FATAL:  DATABASE files are incompatible with server
-- 2020-08-18 10:26:24.838 UTC [1] DETAIL:  The data directory was initialized by PostgreSQL version 12, which is not compatible with this version 10.3 (Debian 10.3-1.pgdg90+1).
-- обязательно чистим pvc

-- 8.0 у меня собрана нормально

-- проблема в версии > 8.0.0 идет добавление по HOSTNAME, а там добавляется в образ хуки на постсоздание
-- и они не видят кластер, а так как контенер без хуков не заканчивает сборку, hostname тоже не получается
 if [ ${POD_IP} ]; then psql --host=citus-master --username=postgres 
 --command="SELECT * FROM master_add_node('${HOSTNAME}.citus-workers', 5432);" ; fi 

-- версия 9.3
-- поэтому сделал версию с заменой HOSTNAME на POD_IP
-- но нужно потом решить проблему со сменой ip - поменять имена нод


*/

-- потраим 10.1-pg12
cd ../citus_10.1pg12
kubectl create -f secrets.yaml
-- версия мастера - старый образ citusdata/citus:7.3.0 
nano master.yaml
kubectl create -f master.yaml
nano workers.yaml
kubectl apply -f workers.yaml

kubectl get all

kubectl exec -it pod/citus-master-5bff7bc99c-hl559 -- bash
psql -U postgres
SELECT * FROM master_get_active_worker_nodes();
-- psql (10.3 (Debian 10.3-1.pgdg90+1))



-- варианты загрузки в citus - внутри контейнера
mkdir /home/1
chmod 777 /home/1
cd /home/1
apt-get update
apt-get install wget
wget https://storage.googleapis.com/postgres13/1000000SalesRecords.csv

psql -U postgres
CREATE TABLE test (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    Region VARCHAR(50),
    Country VARCHAR(50),
    ItemType VARCHAR(50),
    SalesChannel VARCHAR(20),
    OrderPriority VARCHAR(10),
    OrderDate VARCHAR(10),
    OrderID int,
    ShipDate VARCHAR(10),
    UnitsSold int,
    UnitPrice decimal(12,2),
    UnitCost decimal(12,2),
    TotalRevenue decimal(12,2),
    TotalCost decimal(12,2),
    TotalProfit decimal(12,2)
);
-- 2 вариант -- до версии 9.5 (версия citus)
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE TABLE test (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v1(),
    Region VARCHAR(50),
    Country VARCHAR(50),
    ItemType VARCHAR(50),
    SalesChannel VARCHAR(20),
    OrderPriority VARCHAR(10),
    OrderDate VARCHAR(10),
    OrderID int,
    ShipDate VARCHAR(10),
    UnitsSold int,
    UnitPrice decimal(12,2),
    UnitCost decimal(12,2),
    TotalRevenue decimal(12,2),
    TotalCost decimal(12,2),
    TotalProfit decimal(12,2)
);

\timing
SELECT create_distributed_table('test', 'id');
-- ERROR:  function public.uuid_generate_v1() does not exist
kubectl exec -it pod/citus-worker-0 -- psql -U postgres -c 'CREATE EXTENSION IF NOT EXISTS "uuid-ossp";'
kubectl exec -it pod/citus-worker-1 -- psql -U postgres -c 'CREATE EXTENSION IF NOT EXISTS "uuid-ossp";'
kubectl exec -it pod/citus-worker-2 -- psql -U postgres -c 'CREATE EXTENSION IF NOT EXISTS "uuid-ossp";'

copy test (Region,Country,ItemType,SalesChannel,OrderPriority,OrderDate,OrderID,ShipDate,UnitsSold,UnitPrice,UnitCost,TotalRevenue,TotalCost,TotalProfit) FROM '/home/1/1000000SalesRecords.csv' DELIMITER ',' CSV HEADER;
-- или клиент сайд 
-- \copy test (Region,Country,ItemType,SalesChannel,OrderPriority,OrderDate,OrderID,ShipDate,UnitsSold,UnitPrice,UnitCost,TotalRevenue,TotalCost,TotalProfit) FROM '/home/1/1000000SalesRecords.csv' DELIMITER ',' CSV HEADER;

-- вариант с координатором на 10.1
nano master2.yaml
kubectl delete -f master.yaml
kubectl apply -f master2.yaml

kubectl get all

kubectl exec -it pod/citus-master-6cb9c775c4-7sfxq -- bash
psql -U postgres
SELECT * FROM master_get_active_worker_nodes();
SELECT * from master_add_node('citus-worker-0.citus-workers', 5432);
SELECT * from master_add_node('citus-worker-1.citus-workers', 5432);
SELECT * from master_add_node('citus-worker-2.citus-workers', 5432);
SELECT rebalance_table_shards('test');

-- зайдем на сегменты
kubectl exec -it pod/citus-worker-0 -- bash
psql -U postgres
select * from test;
\dt

-- удалим все
kubectl delete all,ing,secrets,pvc,pv --all


-- посмотрим на версию от Алексея
cd ../Alexey10.1pg12
ls -l




gcloud container clusters delete citus --zone us-central1-c
--посмотрим, что осталось от кластера
gcloud compute disks list
----------------------------------------------------------------------------------------------------------------------------







черновик 2
----------------------------------------------------------------------------------------------------------------------------
sudo systemctl enable --now kubelet && systemctl status kubelet

kubectl apply -f https://raw.githubusercontent.com/techarkit/Linux_guides/master/kube-flannel.yml
wget https://raw.githubusercontent.com/techarkit/Linux_guides/master/kube-flannel.yml
yum install git
yum install git
git clone https://github.com/techarkit/Linux_guides.git
git clone https://github.com/techarkit/Linux_guides.git
git clone https://github.com/techarkit/Linux_guides.git

cd Linux_guides
cd Linux_guides
cd Linux_guides

kubectl apply -f kube-flannel.yml


kubectl get nodes
kubectl get nodes
kubectl get nodes


systemctl stop apparmor
systemctl disable apparmor 
systemctl restart containerd.service
systemctl restart kubelet

kubectl -n kube-system apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml

Jul 13 08:25:56 masterkub kubelet: E0713 08:25:56.082419    1576 kubelet.go:2475] "Container runtime network not ready" 

networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network plugin returns error: cni plugin not initialized"


id@machine:/# ip -4 addr show
6: cni0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    inet 10.244.11.1/24 brd 10.244.11.255 scope global cni0
       valid_lft forever preferred_lft forever

ip link delete cni0

----------------------------------------------------------------------------------------------------------------------------

вспомогательные команды
----------------------------------------------------------
kubectl get pods --all-namespaces -o wide
kubectl get pods --all-namespaces -o wide
kubectl get pods --all-namespaces -o wide

#Это предоставит информацию обо всех deployments во всех namespaces
kubectl get deployments --all-namespaces
kubectl get deployments --all-namespaces
kubectl get deployments --all-namespaces

#Это предоставит информацию обо всех deployments в namespaces cert-manager
kubectl get deployments -n cert-manager
kubectl get deployments -n cert-manager
kubectl get deployments -n cert-manager


#Это предоставит информацию обо всех модулях, развертываниях, службах и заданиях в пространстве имен.
kubectl get pods,services,deployments,jobs
kubectl get pods,services,deployments,jobs
kubectl get pods,services,deployments,jobs

#Это предоставит информацию о сервисахесли
kubectl get services
kubectl get services
kubectl get services

#Это предоставит информацию о пространствах имен
kubectl get namespace
kubectl get namespace
kubectl get namespace

#Чтобы перечислить все развертывания:
kubectl get deployments --all-namespaces
kubectl get deployments --all-namespaces
kubectl get deployments --all-namespaces

kubectl get all
kubectl get all
kubectl get all

#показать все поды даже системные
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

#А вот для проверки PV и PVC необходимо выполнить следующую команду :
kubectl apply -f /tmp/citus/postgres-storage.yaml
kubectl get pvc -o wide
kubectl get pv -o wide

kubectl get pv
kubectl describe pv
kubectl get pvc
kubectl describe pvc
kubectl get pv
kubectl get all -o wide

-- посмотрим дефолтный тип стораджа
kubectl get storageclasses

kubectl get pvc -o wide
kubectl get pv -o wide

#удалить все
kubectl delete all,ing,secrets,pvc,pv --all
kubectl delete all,ing,secrets,pvc,pv --all

#прозвонить pod
kubectl describe pod -n kube-system etcd-masterkub.ru-central1.internal
kubectl describe pod -n kube-system etcd-masterkub.ru-central1.internal
kubectl describe pod -n kube-system etcd-masterkub.ru-central1.internal

#логи pod
kubectl logs --namespace kube-system etcd-masterkub.ru-central1.internal
kubectl logs --namespace kube-system etcd-masterkub.ru-central1.internal
kubectl logs --namespace kube-system etcd-masterkub.ru-central1.internal
kubectl logs --namespace kube-system kube-flannel-ds-amd64-5fx2p
kubectl logs --namespace kube-system kube-flannel-ds-amd64-5fx2p
kubectl logs --namespace kube-system kube-flannel-ds-amd64-5fx2p

kubectl logs pod_name(необязательно с -n namespace_name)
kubectl logs pod_name(необязательно с -n namespace_name)
kubectl logs pod_name(необязательно с -n namespace_name)

kubectl logs citus-master-5bff7bc99c-5qw7g

#Запустите команду ниже, чтобы получить события. Это покажет проблему (и все другие события), почему модуль не запланирован.
kubectl get events
kubectl get events
kubectl get events

0/4 nodes are available: pod has unbound immediate PersistentVolumeClaims. preemption: 0/4 nodes are available: 4 No preemption victims found for incoming pod..

---
смотрим интерфейсы
ip -4 addr show
ip -4 addr show
ip -4 addr show

Удалите интерфейс cni0и файлы flannel.1.
sudo ip link delete cni0;
sudo ip link delete flannel.1;
---

tail -f /var/log/messages
tail -f /var/log/messages
tail -f /var/log/messages
----------------------------------------------------------











----------------------------------------------------------------------------------------------------------------------------
требуется удалить ненужные поды

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

------------
[root@kub1 vorori]# kubectl get pods,services,deployments,jobs
NAME                              READY   STATUS    RESTARTS   AGE
pod/citus-0                       0/2     Pending   0          7m50s
pod/citus-coordinator-0           0/2     Pending   0          7m50s
pod/citus-coordinator-replica-0   0/2     Pending   0          7m50s
pod/citus-worker-0                0/2     Pending   0          7m50s

NAME                                TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)             AGE
service/citus                       ClusterIP   None         <none>        5432/TCP,6432/TCP   16h
service/citus-coordinator           ClusterIP   None         <none>        5432/TCP,6432/TCP   16h
service/citus-coordinator-replica   ClusterIP   None         <none>        5432/TCP,6432/TCP   16h
service/citus-worker                ClusterIP   None         <none>        6432/TCP,5432/TCP   16h
service/kubernetes                  ClusterIP   10.96.0.1    <none>        443/TCP             21h
------------

------------
[root@kub1 vorori]# kubectl get services
NAME                        TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)             AGE
citus                       ClusterIP   None         <none>        5432/TCP,6432/TCP   16h
citus-coordinator           ClusterIP   None         <none>        5432/TCP,6432/TCP   16h
citus-coordinator-replica   ClusterIP   None         <none>        5432/TCP,6432/TCP   16h
citus-worker                ClusterIP   None         <none>        6432/TCP,5432/TCP   16h
kubernetes                  ClusterIP   10.96.0.1    <none>        443/TCP             21h
------------

------------
kubectl delete service citus citus-coordinator citus-coordinator-replica citus-worker
kubectl delete service citus citus-coordinator citus-coordinator-replica citus-worker
kubectl delete service citus citus-coordinator citus-coordinator-replica citus-worker
------------

------------
kubectl delete pod citus-0 citus-coordinator-0 citus-coordinator-replica-0 citus-worker-0
kubectl delete pod citus-0 citus-coordinator-0 citus-coordinator-replica-0 citus-worker-0
kubectl delete pod citus-0 citus-coordinator-0 citus-coordinator-replica-0 citus-worker-0
------------

------------
Теперь удалите все объекты, включая OLM, подписки, развертывания, наборы реплик и т. в namespace openshift-submariner:
$ kubectl delete olm,svc,rs,rc,subs,deploy,jobs,pods --all -n openshift-submariner
$ kubectl delete olm,svc,rs,rc,subs,deploy,jobs,pods --all -n openshift-submariner
$ kubectl delete olm,svc,rs,rc,subs,deploy,jobs,pods --all -n openshift-submariner
------------

------------
kubectl delete deploy,pods --all -n cert-manager
deployment.apps "cert-manager" deleted
deployment.apps "cert-manager-cainjector" deleted
deployment.apps "cert-manager-webhook" deleted
pod "cert-manager-6cf96f6d7f-x6c7c" deleted
pod "cert-manager-cainjector-5fcd65b864-7gdvl" deleted
pod "cert-manager-webhook-5bdcd8f47c-4s9lm" deleted
------------


----------------------------------------------------------------------------------------------------------------------------

черновик 1:
----------------------------------------------------------------------------------------------------------------------------
если надо сделать если вы уже создали предыдущий кластер
kubeadm reset
kubeadm reset
kubeadm reset



kubeadm init
kubeadm init --apiserver-advertise-address=10.129.0.36 --pod-network-cidr=192.168.0.0/16
kubeadm init --apiserver-advertise-address=10.129.0.12
kubeadm init --apiserver-advertise-address=10.129.0.12
kubeadm init --apiserver-advertise-address=10.129.0.12

Я использовал следующую команду для создания конфигурации по умолчанию (при новой установке на master):
kubeadm init --apiserver-advertise-address=10.129.0.18
kubeadm init --apiserver-advertise-address=10.129.0.18
kubeadm init --apiserver-advertise-address=10.129.0.18

kubeadm init --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=master_nodeIP
kubeadm init --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=10.129.0.12

kubeadm init --pod-network-cidr="10.10.0.0/16" --apiserver-advertise-address="10.128.0.20"
kubeadm init --pod-network-cidr="10.10.0.0/16" --apiserver-advertise-address="10.128.0.20"
или
kubeadm init phase kubelet-start
kubeadm init phase kubelet-start

где:
--pod-network-cidr=10.10.0.0/16 - диапазон сети pod
--apiserver-advertise-address=master_nodeIP  - ip адрес кластера(главного узла кластера) вставляем сюда ip VM на которой инициализируем кластер

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

kubectl get nodes
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
----------------------------------------------------------------------------------------------------------------------------




1
добавил метки 
kubectl label nodes masterkub.ru-central1.internal disktype=citusmaster
kubectl label nodes kub1.ru-central1.internal disktype=citusworker1
kubectl label nodes kub2.ru-central1.internal disktype=citusworker2
kubectl label nodes kub3.ru-central1.internal disktype=citusworker3

citusmaster
citusworker1
citusworker2
citusworker3


[root@masterkub vorori]# kubectl get nodes --show-labels
NAME                             STATUS   ROLES           AGE     VERSION   LABELS
kub1.ru-central1.internal        Ready    <none>          2d6h    v1.26.1   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,disktype=citusworker1,kubernetes.io/arch=amd64,kubernetes.io/hostname=kub1.ru-central1.internal,kubernetes.io/os=linux
kub2.ru-central1.internal        Ready    <none>          2d5h    v1.26.1   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,disktype=citusworker2,kubernetes.io/arch=amd64,kubernetes.io/hostname=kub2.ru-central1.internal,kubernetes.io/os=linux
kub3.ru-central1.internal        Ready    <none>          2d5h    v1.26.1   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,disktype=citusworker3,kubernetes.io/arch=amd64,kubernetes.io/hostname=kub3.ru-central1.internal,kubernetes.io/os=linux
masterkub.ru-central1.internal   Ready    control-plane   2d10h   v1.26.1   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,disktype=citusmaster,kubernetes.io/arch=amd64,kubernetes.io/hostname=masterkub.ru-central1.internal,kubernetes.io/os=


2)
создал директорию для мастера
mkdir /var/pgsql-volume-master
mkdir /var/pgsql-volume-master
mkdir /var/pgsql-volume-master


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



3
[root@masterkub citus]# kubectl apply -f /tmp/citus/mypv.yaml
persistentvolume/storage-citusworker1 created

[root@masterkub citus]# kubectl get pv
NAME                   CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
storage-citusworker1   10Gi       RWO            Retain           Available                                   12s                         44s

4


5
vim /tmp/citus/secrets.yaml


apiVersion: v1
kind: Secret
metadata:
  name: citus-secrets
type: Opaque
data:
  password: MjMzODQ4NA==

kubectl create -f /tmp/citus/secrets.yaml

[root@masterkub citus]# [root@masterkub citus]# kubectl create -f /tmp/citus/secrets.yaml
secret/citus-secrets created



6
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


7
kubectl apply -f /tmp/citus/pvcmaster.yaml
kubectl get pvc -o wide

[root@masterkub citus]# [root@masterkub citus]# kubectl apply -f /tmp/citus/pvcmaster.yaml
persistentvolumeclaim/citus-master-pvc created
[root@masterkub citus]# kubectl get pvc -o wide
NAME               STATUS   VOLUME                 CAPACITY   ACCESS MODES   STORAGECLASS   AGE   VOLUMEMODE
citus-master-pvc   Bound    storage-citusworker1   10Gi       RWO                           12s   Filesystem



8
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
			
9			
kubectl apply -f /tmp/citus/master.yaml


10
[root@masterkub citus]# kubectl get pods
NAME                            READY   STATUS    RESTARTS   AGE
citus-master-5bff7bc99c-kd89l   1/1     Running   0          41s


11
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


12

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

kubectl apply -f /tmp/citus/worker.yaml


13 тут какая то хрень походу надо пределать основной диск на мастере мастеров чтобы все ноды работали
[root@masterkub citus]# [root@masterkub citus]# kubectl apply -f /tmp/citus/worker.yaml
service/citus-workers created
statefulset.apps/citus-worker created
[root@masterkub citus]# kubectl get pods
NAME                            READY   STATUS    RESTARTS   AGE
citus-master-5bff7bc99c-kd89l   1/1     Running   0          2m32s
citus-worker-0                  1/1     Running   0          14s
citus-worker-1                  1/1     Running   0          11s
citus-worker-2                  1/1     Running   0          9s


14
kubectl exec -it pod/citus-master-5bff7bc99c-kd89l -- bash
psql -U postgres
postgres=# SELECT * FROM master_get_active_worker_nodes();
          node_name           | node_port
------------------------------+-----------
 citus-worker-2.citus-workers |      5432
 citus-worker-0.citus-workers |      5432
 citus-worker-1.citus-workers |      5432
(3 rows)







15
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



SELECT create_distributed_table('test', 'id');
-- ERROR:  function public.uuid_generate_v1() does not exist
kubectl exec -it pod/citus-worker-0 -- psql -U postgres -c 'CREATE EXTENSION IF NOT EXISTS "uuid-ossp";'
kubectl exec -it pod/citus-worker-1 -- psql -U postgres -c 'CREATE EXTENSION IF NOT EXISTS "uuid-ossp";'
kubectl exec -it pod/citus-worker-2 -- psql -U postgres -c 'CREATE EXTENSION IF NOT EXISTS "uuid-ossp";'