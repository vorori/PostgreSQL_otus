#### 1)

#### Развернуть Постгрес на ВМ

<pre>
создал vm centos 

sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
sudo yum -y repolist
sudo yum install postgresql15-contrib

sudo /usr/pgsql-15/bin/postgresql-15-setup initdb
systemctl enable postgresql-15.service
systemctl status postgresql-15.service
systemctl restart postgresql-15.service
</pre>

#### 2)

#### Протестировать pg_bench

<pre>

</pre>

#### 3)

#### Выставить оптимальные настройки

<pre>

</pre>

#### 4)

####  Проверить насколько выросла производительность


<pre>

</pre>


#### 5)

####  Настроить кластер на оптимальную производительность не обращая внимания на стабильность БД


<pre>

</pre>

