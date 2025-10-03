PostgreSQL Кластер
=========
Схема кластера 
![img.png](img.png) 
Нам понадобится 4 виртуальные машины: 


| NodeName |  IP  | Role     |
|:---------|:-----------:|----------------------|
| node1 | 192.168.5.1 | postgresql + patroni |
| node2 | 192.168.5.2 | postgresql + patroni |
| etcd  | 192.168.5.3 | etcd     |
| haproxy | 192.168.5.4 | haproxy    |


## Настройка на нодах 1 и 2

Установка необходимых пакетов
```bash
sudo apt update

sudo apt install postgresql python3-pip

sudo pip3 install --upgrade pip

sudo pip3 install psycopg2 patroni

```

**Настройка patroni** 

Создадим файл с конфигом 

```
sudo nano /etc/patroni.yml
```
Задать параметры
```
scope: postgres
namespace: /db/
name: cluster

restapi:
 listen: <nodeN_ip>:8008
 connect_address: <nodeN_ip>:8008

etcd:
 host: <etcdnode_ip>:2379

bootstrap:
 dcs:
 ttl: 30
 loop_wait: 10
 retry_timeout: 10
 maximum_lag_on_failover: 1048576
 postgresql:
  use_pg_rewind: true
  use_slots: true
  parameters:

 initdb:
 - encoding: UTF8
 - data-checksums

 pg_hba:
 - host replication replicator 127.0.0.1/32 md5
 - host replication replicator <node1_ip>/0 md5
 - host replication replicator <node2_ip>/0 md5
 - host all all 0.0.0.0/0 md5

 users:
 admin:
  password: admin
  options:
  - createrole
  - createdb

postgresql:
 listen: <nodeN_ip>:5432
 connect_address: <nodeN_ip>:5432
 data_dir: /var/lib/postgres/11/cluster
 pgpass: /tmp/pgpass
 authentication:
 replication:
  username: replicator
  password: ************
 superuser:
  username: postgres
  password: ************
 parameters:
  unix_socket_directories: '.'

tags:
 nofailover: false
 noloadbalance: false
 clonefrom: false
 nosync: false
```

Создадим необходимые для работы папки

```
sudo mkdir -p /var/lib/postgres/11/cluster

sudo chown postgres:postgres /var/lib/postgres/11/cluster

sudo chmod 700 /var/lib/postgres/11/cluster
```

Создадим файл-сервис для patroni

```
sudo nano /etc/systemd/system/patroni.service
```

Вставим туда текст

```
[Unit]
Description=High availability PostgreSQL Cluster
After=syslog.target network.target

[Service]
Type=simple
User=postgres
Group=postgres
ExecStart=/usr/local/bin/patroni /etc/patroni.yml
KillMode=process
TimeoutSec=30
Restart=no

[Install]
WantedBy=multi-user.targ
```

Запуск patroni

```
sudo systemctl daemon-reload

sudo systemctl start patroni

sudo systemctl status patroni
```

## Настройка etcd ноды

Так как etcd на AstraLinux находится в extended репозитории, необходимо его добавить

```
sudo nano /etc/apt/sources.list
```

Вставляем туда

```
deb http://download.astralinux.ru/astra/stable/1.7_x86-64/repository-extended/ 1.7_x86-64 main contrib non-free
```

Установка пакета
```
sudo apt update
sudo apt install etcd -y
```

Другой вариант - заранее выкачать .deb пакет etcd и установить его командой

```
sudo dpkg -i name_of_etcd_package.deb
```

Конфигурация etcd

```commandline
sudo nano /etc/default/etcd
```

```
ETCD_LISTEN_PEER_URLS="http://<etcdnode_ip>:2380"
ETCD_LISTEN_CLIENT_URLS="http://localhost:2379,http://<etcdnode_ip>:2379"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://<etcdnode_ip>:2380"
ETCD_INITIAL_CLUSTER="default=http://<etcdnode_ip>:2380,"
ETCD_ADVERTISE_CLIENT_URLS="http://<etcdnode_ip>:2379"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER_STATE="new"
```

Запуск etcd
```
sudo systemctl restart etcd

sudo systemctl status etcd
```

Проверка работы

```
curl http://<etcdnode_ip>:2380/members
```

## Настройка HAProxy

Так как etcd на AstraLinux находится в base репозитории, необходимо его добавить

```
sudo nano /etc/apt/sources.list
```

Вставляем туда

```
deb http://download.astralinux.ru/astra/stable/1.7_x86-64/repository-base/ 1.7_x86-64 main contrib non-free
```

Настройка

```
sudo nano /etc/haproxy/haproxy.cfg
```

Вставить в файл:
```
global

  maxconn 100
  log  127.0.0.1 local2

defaults
  log global
  mode tcp
  retries 2
  timeout client 30m
  timeout connect 4s
  timeout server 30m
  timeout check 5s

listen stats
 mode http
 bind *:7000
 stats enable
 stats uri /

listen postgres
 bind *:5000
 option httpchk
 http-check expect status 200
 default-server inter 3s fall 3 rise 2 on-marked-down shutdown-sessions
 server node1 <node1_ip>:5432 maxconn 100 check port 8008
 server node2 <node2_ip>:5432 maxconn 100 check port 8008
```

Запуск

```
sudo systemctl restart haproxy

sudo systemctl status haproxy
```

Проверка работы

```
http://<haproxynode_ip>:7000/>
```

**Веб-интерфейс будет показывать мастер-ноду как _UP_, а остальные ноды - _DOWN_**

## Проверка работы

Остановим сервис patroni на одной из нод кластера

```
sudo systemctl stop patroni
```

До:

![img_1.png](img_1.png)

После:

![img_2.png](img_2.png)

## Подключение к кластеру

В качестве базы данных у программы указывать IP адрес ноды с HAProxy с 5000 портом.