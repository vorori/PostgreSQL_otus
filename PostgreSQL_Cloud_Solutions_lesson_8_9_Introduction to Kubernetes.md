#### 1)

#### Устанавливаем minikube

<pre>
доп мануал
https://ostechnix.com/install-kubernetes/
https://ostechnix.com/install-kubernetes/
https://ostechnix.com/install-kubernetes/


1.1)
Install Docker

vi /etc/yum.repos.d/docker.repo

[docker]
baseurl=https://download.docker.com/linux/centos/7/x86_64/stable/
gpgcheck=0


yum repolist
yum -y install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
systemctl start docker
systemctl enable docker
systemctl status docker

1.2)
Install Conntrack
Conntrack является частью платформы Netlifier. Для хорошей работы сложной сети Kubernetes требуется, 
поскольку узлы должны отслеживать соединения между тысячами модулей и сервисов.

yum -y install conntrack
yum -y install conntrack
yum -y install conntrack

1.3)
Install Kubernetes Client (Kubectl)
Kubectl — это инструмент командной строки для работы с Kubernetes. Вы можете скачать kubectl с помощью следующей команды

curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl


выдаем права
chmod +x kubectl


перемешаем бинарник в /usr/local/bin/
mv kubectl /usr/local/bin/


Проверяем установку, проверив версию kubeclt:
kubectl version --client -o json


{
  "clientVersion": {
    "major": "1",
    "minor": "27",
    "gitVersion": "v1.27.2",
    "gitCommit": "7f6f68fdabc4df88cfea2dcf9a19b2b830f1e647",
    "gitTreeState": "clean",
    "buildDate": "2023-05-17T14:20:07Z",
    "goVersion": "go1.20.4",
    "compiler": "gc",
    "platform": "linux/amd64"
  },
  "kustomizeVersion": "v5.0.1"
}


1.4)
Install Minikube
Загрузите пакет minicube с помощью команды:
wget https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64


Дайте разрешение на выполнение пакету minicube:
chmod +x minikube-linux-amd64


переместите пакет Minikube в /usr/local/bin:
mv minikube-linux-amd64 /usr/local/bin/minikube

1.5)
проверяем версию
minikube version
minikube version: v1.30.1
commit: 08896fd1dc362c097c925146c4a0d0dac715ace0

1.6)
запуск
minikube start
minikube start
minikube start

[root@localhost tmp]# minikube start --force
* minikube v1.30.1 on Centos 7.9.2009 (hyperv/amd64)
! minikube skips various validations when --force is supplied; this may lead to unexpected behavior
* Using the docker driver based on existing profile
* The "docker" driver should not be used with root privileges. If you wish to continue as root, use --force.
* If you are running minikube within a VM, consider using --driver=none:
*   https://minikube.sigs.k8s.io/docs/reference/drivers/none/
* Tip: To remove this root owned cluster, run: sudo minikube delete
* Starti1.3ng control plane node minikube in cluster minikube
* Pulling base image ...
* Another minikube instance is downloading dependencies...
* Downloading Kubernetes v1.26.3 preload ...
    > preloaded-images-k8s-v18-v1...:  397.02 MiB / 397.02 MiB  100.00% 1.32 Mi
    > index.docker.io/kicbase/sta...:  373.53 MiB / 373.53 MiB  100.00% 758.93
! minikube was unable to download gcr.io/k8s-minikube/kicbase:v0.0.39, but successfully downloaded docker.io/kicbase/stable:v0.0.39 as a fallback image
* Creating docker container (CPUs=2, Memory=2200MB) ...
* Preparing Kubernetes v1.26.3 on Docker 23.0.2 ...
  - Generating certificates and keys ...
  - Booting up control plane ...
  - Configuring RBAC rules ...
* Configuring bridge CNI (Container Networking Interface) ...
  - Using image gcr.io/k8s-minikube/storage-provisioner:v5
* Verifying Kubernetes components...
* Enabled addons: default-storageclass, storage-provisioner
* Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default


1.7)
проверяем статус 
[root@localhost ~]# minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured

1.8)
статус и роли узлов с помощью команды kubectl:
[root@localhost ~]# kubectl get nodes
NAME       STATUS   ROLES           AGE    VERSION
minikube   Ready    control-plane   2m5s   v1.26.3


1.9)
доступ к панели управления пользовательского интерфейса Kubernetes.
Чтобы получить доступ к панели инструментов Kubernetes через веб-браузер, запустите:
minikube dashboard --url

* Enabling dashboard ...
  - Using image docker.io/kubernetesui/dashboard:v2.7.0
  - Using image docker.io/kubernetesui/metrics-scraper:v1.0.8
* Some dashboard features require the metrics-server addon. To enable all features please run:

        minikube addons enable metrics-server


* Verifying dashboard health ...
* Launching proxy ...
* Verifying proxy health ...
http://127.0.0.1:36408/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/


1.10)
но по этому адресу естесственно я недостукиваюсь с моей windows тачки
http://127.0.0.1:36408/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/


1.11)
--поэтому добавляю доступ на фаерволе для моей сети
sudo firewall-cmd --zone=public --add-port=8001/tcp --permanent
sudo firewall-cmd --reload


1.12)
--и проксирую кубер на порт 8001
kubectl proxy --address='0.0.0.0' --disable-filter=true
W0605 12:22:01.709591   24664 proxy.go:175] Request filter disabled, your proxy is vulnerable to XSRF attacks, please be cautious
Starting to serve on [::]:8001


1.12)
теперь кубер доступен по Ip из моей сети счастье пришло
http://192.168.7.37:8001/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/#/workloads?namespace=default
http://192.168.7.37:8001/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/#/workloads?namespace=default
http://192.168.7.37:8001/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/#/workloads?namespace=default


</pre>


#### 2)

#### Разворачиваем PostgreSQL 14 через манифест

<pre>

2.1)
редактирую postgres.yaml нам нужна версия 14 postgres
kubectl apply -f postgres.yaml

2.2)
minikube service postgres --url pg-kub2
minikube service postgres --url pg-kub2
http://192.168.49.2:30849


2.3)
в дашборде появился postgre
postgres
app: postgres
NodePort
10.100.148.12
postgres:5432 TCP
postgres:30849 TCP
-
33 minutes ago


2.4)
показать все сервисы котрые есть в кубике
[root@localhost ~]# kubectl get all
NAME                         READY   STATUS    RESTARTS   AGE
pod/postgres-statefulset-0   1/1     Running   0          36m

NAME               TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/postgres   NodePort   10.109.191.49   <none>        5432:30253/TCP   38m

NAME                                    READY   AGE
statefulset.apps/postgres-statefulset   1/1     36m

2.5)
устанавливаю клиент psql и подключаюсь к postgres
sudo yum install postgresql14

2.6)
пробросил порт Postgre
[root@localhost ~]# kubectl port-forward service/postgres 5432:5432
Forwarding from 127.0.0.1:5432 -> 5432
Forwarding from [::1]:5432 -> 5432

2.7)
подключился к кластеру
[root@localhost ~]# psql -U myuser -h 127.0.0.1 -p 5432 -d postgres
psql (14.8, server 14.8 (Debian 14.8.pgdg110+1))
WARNING: psql major version 14, server major version 15.
         Some psql features might not work.
Type "help" for help.

postgres=# \l
                              List of databases
   Name    | Owner  | Encoding |  Collate   |   Ctype    | Access privileges
-----------+--------+----------+------------+------------+-------------------
 myapp     | myuser | UTF8     | en_US.utf8 | en_US.utf8 |
 postgres  | myuser | UTF8     | en_US.utf8 | en_US.utf8 |
 template0 | myuser | UTF8     | en_US.utf8 | en_US.utf8 | =c/myuser        +
           |        |          |            |            | myuser=CTc/myuser
 template1 | myuser | UTF8     | en_US.utf8 | en_US.utf8 | =c/myuser        +
           |        |          |            |            | myuser=CTc/myuser
(4 rows)

postgres=#


2.8)
создал тестовые данные
postgres=# CREATE TABLE client(id serial, name text);
CREATE TABLE
postgres=# INSERT INTO client(name) values('Ivan');
INSERT 0 1
postgres=#

</pre>


#### 3)

#### Задание повышенной сложности* Разворачиваем PostgreSQL 14 с помощью helm


<pre>
доп мануал
https://phoenixnap.com/kb/postgresql-kubernetes
https://phoenixnap.com/kb/postgresql-kubernetes
https://phoenixnap.com/kb/postgresql-kubernetes


https://stackoverflow.com/questions/55973901/how-can-i-list-all-available-charts-under-a-helm-repo#:~:text=Simply%20helm%20search%20repo%20to,on%20the%20input%20search%20text.
https://stackoverflow.com/questions/55973901/how-can-i-list-all-available-charts-under-a-helm-repo#:~:text=Simply%20helm%20search%20repo%20to,on%20the%20input%20search%20text.
https://stackoverflow.com/questions/55973901/how-can-i-list-all-available-charts-under-a-helm-repo#:~:text=Simply%20helm%20search%20repo%20to,on%20the%20input%20search%20text.


https://github.com/bitnami/charts/tree/main/bitnami/postgresql
https://github.com/bitnami/charts/tree/main/bitnami/postgresql



3.1)
добавляю репозиторий Helm
Helm add repo!!!!!!!!!!!!!!!!!!!
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add bitnami https://charts.bitnami.com/bitnami


[root@localhost rrrrr]# helm repo add bitnami https://charts.bitnami.com/bitnami
"bitnami" has been added to your repositories


3.2)
После добавления репозитория обновите локальные репозитории.
helm repo update


[root@localhost rrrrr]# helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "bitnami" chart repository
Update Complete. ⎈Happy Helming!⎈


3.3)
Почистим стары postge кластера удалим все лишнее
[root@localhost postgres]# kubectl delete -f postgres.yaml
service "postgres" deleted
statefulset.apps "postgres-statefulset" deleted

kubectl delete -n default service postgres
kubectl delete -n default service postgres
kubectl delete -n default service postgres


3.4)
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


3.6)
СОЗДАТЬ и ПРИМЕНИТЬ том постоянного хранилища
Данные в вашей базе данных Postgres должны сохраняться при перезапуске модуля.

Для этого создайте ресурс PersistentVolume в файле YAML 
vim postgres-pv.yaml

Содержимое файла определяет:
--Сам ресурс.
--Класс хранения.
--Объем выделенного хранилища.
--Режимы доступа.
--Путь монтирования в хост-системе.
--В этом примере используется следующая конфигурация:

apiVersion: v1
kind: PersistentVolume
metadata:
  name: postgresql-pv
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"


3.7)
применяем конфигурацию с помощью kubectl:
kubectl apply -f postgres-pv.yaml


[root@localhost tmp]# kubectl apply -f postgres-pv.yaml
persistentvolume/postgresql-pv created


3.8)
СОЗДАТЬ Persistent Volume Claim (PVC)
Создайте Persistent Volume Claim (PVC), чтобы запросить хранилище, выделенное на предыдущем шаге.

vim postgres-pvc.yaml

В примере используется следующая конфигурация:

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgresql-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi


3.9)
Сохраните файл. Применить конфигурацию с помощью kubectl :
kubectl apply -f postgres-pvc.yaml

3.10)
Система подтверждает успешное создание PVC.
[root@localhost tmp]# kubectl apply -f postgres-pvc.yaml
persistentvolumeclaim/postgresql-pv-claim created


3.11)
Столбец состояния показывает, что заявка имеет статус Bound .
[root@localhost tmp]# kubectl get pvc
NAME                               STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
postgredb-postgres-statefulset-0   Bound    pvc-c27e0117-0079-4520-871e-f5638f9f0c9d   1Gi        RWO            standard       105m
postgresql-pv-claim                Bound    postgresql-pv                              10Gi       RWO            manual         66s


3.12)
Установи Helm Chart
Установи рулевую диаграмму с помощью helm install команды. 
Добавь --set флаги в команду для подключения установки к созданному вами PVC и включите разрешения тома:

вот такая штука нужна была чтобы избежать ошибок
export HELM_EXPERIMENTAL_OCI=1
export HELM_EXPERIMENTAL_OCI=1
export HELM_EXPERIMENTAL_OCI=1


3.13)
сама команда которая далась потом и кровью
helm install app-prod bitnami/postgresql --set persistence.existingClaim=pvc-c27e0117-0079-4520-871e-f5638f9f0c9d --set volumePermissions.enabled=true
helm install app-prod bitnami/postgresql --set persistence.existingClaim=pvc-c27e0117-0079-4520-871e-f5638f9f0c9d --set volumePermissions.enabled=true
helm install app-prod bitnami/postgresql --set persistence.existingClaim=pvc-c27e0117-0079-4520-871e-f5638f9f0c9d --set volumePermissions.enabled=true

Система отобразит отчет об успешной установке.
УРААААААААААААААААААААААААААААААААААААААААА!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
[root@localhost rrrrr]# helm install app-prod bitnami/postgresql --set persistence.existingClaim=pvc-c27e0117-0079-4520-871e-f5638f9f0c9d --set volumePermissions.enabled=true
NAME: app-prod
LAST DEPLOYED: Mon Jun  5 18:29:06 2023
NAMESPACE: pg-kub1
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: postgresql
CHART VERSION: 12.5.6
APP VERSION: 15.3.0

** Please be patient while the chart is being deployed **

PostgreSQL can be accessed via port 5432 on the following DNS names from within your cluster:

    app-prod-postgresql.pg-kub1.svc.cluster.local - Read/Write connection

To get the password for "postgres" run:

    export POSTGRES_PASSWORD=$(kubectl get secret --namespace pg-kub1 app-prod-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)

To connect to your database run the following command:

    kubectl run app-prod-postgresql-client --rm --tty -i --restart='Never' --namespace pg-kub1 --image docker.io/bitnami/postgresql:15.3.0-debian-11-r7 --env="PGPASSWORD=$POSTGRES_PASSWORD" \
      --command -- psql --host app-prod-postgresql -U postgres -d postgres -p 5432

    > NOTE: If you access the container using bash, make sure that you execute "/opt/bitnami/scripts/postgresql/entrypoint.sh /bin/bash" in order to avoid the error "psql: local user with ID 1001} does not exist"

To connect to your database from outside the cluster execute the following commands:

    kubectl port-forward --namespace pg-kub1 svc/app-prod-postgresql 5432:5432 &
    PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U postgres -d postgres -p 5432

WARNING: The configured password will be ignored on new installation in case when previous Posgresql release was deleted through the helm command. In that case, old PVC will have an old password, and setting it through helm won't take effect. Deleting persistent volumes (PVs) will solve the issue.



3.14)
зашел в дашборд а там создается инфра все ок
в NAMESPACE ----> pg-kub1

\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\
Events
Name
Reason
Message
Source
Sub-object
Count
First Seen
Last Seen
app-prod-postgresql-0.1765cd0bd77e7340
Started
Started container postgresql
kubelet minikube
spec.containers{postgresql}

6 minutes ago
6 minutes ago
app-prod-postgresql-0.1765cd0ba66da924
Pulled
Successfully pulled image "docker.io/bitnami/postgresql:15.3.0-debian-11-r7" in 1m37.1449869s (1m37.1449951s including waiting)
kubelet minikube
spec.containers{postgresql}

6 minutes ago
6 minutes ago
app-prod-postgresql-0.1765cd0bc3efef48
Created
Created container postgresql
kubelet minikube
spec.containers{postgresql}

6 minutes ago
6 minutes ago
app-prod-postgresql-0.1765ccf508224eac
Pulling
Pulling image "docker.io/bitnami/postgresql:15.3.0-debian-11-r7"
kubelet minikube
spec.containers{postgresql}

7 minutes ago
7 minutes ago
app-prod-postgresql-0.1765ccf4cd8c2ccc
Created
Created container init-chmod-data
kubelet minikube
spec.initContainers{init-chmod-data}

7 minutes ago
7 minutes ago
app-prod-postgresql-0.1765ccf4d9e3098c
Started
Started container init-chmod-data
kubelet minikube
spec.initContainers{init-chmod-data}

7 minutes ago
7 minutes ago
app-prod-postgresql-0.1765ccf4afaf4004
Pulled
Successfully pulled image "docker.io/bitnami/bitnami-shell:11-debian-11-r120" in 37.155415s (37.155471s including waiting)
kubelet minikube
spec.initContainers{init-chmod-data}

7 minutes ago
7 minutes ago
app-prod-postgresql-0.1765ccec090b5ca4
Pulling
Pulling image "docker.io/bitnami/bitnami-shell:11-debian-11-r120"
kubelet minikube
spec.initContainers{init-chmod-data}

8 minutes ago
8 minutes ago
app-prod-postgresql-0.1765ccebc47132f8
Scheduled
Successfully assigned pg-kub1/app-prod-postgresql-0 to minikube
default-scheduler

8 minutes ago
8 minutes ago
\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\


3.15)
Подключитесь к клиенту PostgreSQL.
Экспортирую POSTGRES_PASSWORDпеременную среды, чтобы иметь возможность войти в экземпляр PostgreSQL:

export POSTGRES_PASSWORD=$(kubectl get secret --namespace pg-kub1 app-prod-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)
export POSTGRES_PASSWORD=$(kubectl get secret --namespace pg-kub1 app-prod-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)
export POSTGRES_PASSWORD=$(kubectl get secret --namespace pg-kub1 app-prod-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)


3.16)
[root@localhost ~]# kubectl get all
NAME                             READY   STATUS    RESTARTS   AGE
pod/app-prod-postgresql-0        1/1     Running   0          17m
pod/app-prod-postgresql-client   0/1     Error     0          7m26s

NAME                             TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
service/app-prod-postgresql      ClusterIP   10.104.71.94   <none>        5432/TCP   17m
service/app-prod-postgresql-hl   ClusterIP   None           <none>        5432/TCP   17m

NAME                                   READY   AGE
statefulset.apps/app-prod-postgresql   1/1     17m



3.17)
открываем в другом окне терминала перенаправление портов , чтобы перенаправить порт Postgres:
kubectl port-forward --namespace pg-kub1 service/app-prod-postgresql  5432:5432

[root@localhost ~]# kubectl port-forward --namespace pg-kub1 service/app-prod-postgresql  5432:5432
Forwarding from 127.0.0.1:5432 -> 5432
Forwarding from [::1]:5432 -> 5432


3.18)
подключения к psql, клиенту PostgreSQL:

PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U postgres -d postgres -p 5432

[root@localhost rrrrr]# PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U postgres -d postgres -p 5432
psql (14.8, server 15.3)
WARNING: psql major version 14, server major version 15.
         Some psql features might not work.
Type "help" for help.

postgres=# \д
invalid command \д
Try \? for help.
postgres=# \l
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges
-----------+----------+----------+-------------+-------------+-----------------------
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
(3 rows)

postgres=#

</pre>
