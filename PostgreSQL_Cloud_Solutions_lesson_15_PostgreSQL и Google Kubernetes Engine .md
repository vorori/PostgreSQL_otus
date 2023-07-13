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
----------------------------------------------------------------------------------------------------------------------------

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

----------------------------------------------------------------------------------------------------------------------------
Pull the images , извлекает образы для версии Kubernetes 1.26.
sudo kubeadm config images pull --image-repository=registry.k8s.io --cri-socket unix:///run/containerd/containerd.sock --kubernetes-version v1.26.1
sudo kubeadm config images pull --image-repository=registry.k8s.io --cri-socket unix:///run/containerd/containerd.sock --kubernetes-version v1.26.1
sudo kubeadm config images pull --image-repository=registry.k8s.io --cri-socket unix:///run/containerd/containerd.sock --kubernetes-version v1.26.1
----------------------------------------------------------------------------------------------------------------------------

----------------------------------------------------------------------------------------------------------------------------
запускаю команду инициализации kubeadm на узле плоскости управления.
https://blog.kubesimplify.com/kubernetes-126
https://blog.kubesimplify.com/kubernetes-126
https://blog.kubesimplify.com/kubernetes-126
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


kubectl get all
kubectl get all
kubectl get all

#показать все поды даже системные
kubectl get all -A
kubectl get all -A
kubectl get all -A

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
если надо сделать если вы уже создали предыдущий кластер
kubeadm reset
kubeadm reset
kubeadm reset


черновик:
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
















