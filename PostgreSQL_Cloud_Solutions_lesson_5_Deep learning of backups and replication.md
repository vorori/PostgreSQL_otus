#### 1)

#### Делам бэкап Постгреса используя WAL-G или pg_probackup и восстанавливаемся на другом кластере



#### 1.1)
<pre>
создал vm centos 4 core 4gb озу

sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
sudo yum -y repolist
sudo yum install postgresql15-contrib

sudo /usr/pgsql-15/bin/postgresql-15-setup initdb
systemctl enable postgresql-15.service
systemctl status postgresql-15.service
systemctl restart postgresql-15.service
</pre>

#### 2)

#### Задание повышенной сложности* под нагрузкой* бэкап снимаем с реплики**

<pre>

</pre>

