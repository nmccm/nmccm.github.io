---
title: "MariaDB Replication 기반 이중화(복제)"
categories: 
- Docker
- MariaDB
tags:
- Docker
- MariaDB
- Replication
---
복제는 하나 이상의 서버(기본 서버라고 함)의 내용을 하나 이상의 서버(복제본 서버라고 함)에 미러링할 수 있는 기능입니다. 어떤 데이터를 복제할지 제어할 수 있습니다. 모든 데이터베이스, 하나 이상의 데이터베이스 또는 데이터베이스 내의 테이블은 각각 선택적으로 복제될 수 있습니다. 복제에 사용되는 주요 메커니즘은 바이너리 로그 입니다. 바이너리 로깅이 활성화된 경우, 데이터베이스에 대한 모든 업데이트(데이터 조작 및 데이터 정의)가 바이너리 로그 이벤트로 바이너리 로그에 기록됩니다. 복제본은 복제할 데이터에 액세스하기 위해 각 기본에서 바이너리 로그를 읽습니다. 바이너리 로그와 동일한 형식을 사용하여 복제본에 릴레이 로그가 생성되고, 이를 사용하여 복제를 수행합니다. 더 이상 필요하지 않으면 이전 릴레이 로그 파일이 제거됩니다.
> Replication is a feature allowing the contents of one or more servers (called primaries) to be mirrored on one or more servers (called replicas). You can exert control over which data to replicate. All databases, one or more databases, or tables within a database can each be selectively replicated. The main mechanism used in replication is the binary log. If binary logging is enabled, all updates to the database (data manipulation and data definition) are written into the binary log as binlog events. Replicas read the binary log from each primary in order to access the data to replicate. A relay log is created on the replica, using the same format as the binary log, and this is used to perform the replication. Old relay log files are removed when no longer needed. A replica server keeps track of the position in the primary's binlog of the last event applied on the replica. This allows the replica server to re-connect and resume from where it left off after replication has been temporarily stopped. It also allows a replica to disconnect, be cloned and then have the new replica resume replication from the same primary. Primaries and replicas do not need to be in constant communication with each other. It's quite possible to take servers offline or disconnect from the network, and when they come back, replication will continue where it left off.  

### 0. 환경
> Window Hyper-V 10.0.22621.1  
> Ubuntu 24.04 LTS  
> MariaDB 11.4.2


### 1. Init
a) 호스트 시스템 업데이트  
b) 아이피 확인을 위한 도구 설치 (net-tools)  
c) 도커 서비스 시작  
d) 도커 서비스 시작 여부 확인  
e) 마리아 디비 도커 이미지 다운로드  
f) 도커 이미지 정상 다운로드 여부 확인    
g) 마리아 디비에서만 사용될 네트워크 생성  
h) 네트워크 설정 여부 확인  
```bash
$ sudo apt update
$ sudo apt install net-tools
$ sudo systemctl restart docker

$ sudo ps -ef | grep docker
root         878       1  0 01:07 ?        00:00:01 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
devteam+    3170    3160  0 01:43 pts/2    00:00:00 grep --color=auto docker

$ sudo docker pull mariadb:latest
latest: Pulling from library/mariadb
9c704ecd0c69: Already exists
f8f7f3c9a741: Pull complete
97c034108521: Pull complete
50366979c20a: Pull complete
0221331af5c0: Pull complete
e3c4d1c9d9cb: Pull complete
eef7a8467f98: Pull complete
60c15bb5bb03: Pull complete
Digest: sha256:e59ba8783bf7bc02a4779f103bb0d8751ac0e10f9471089709608377eded7aa8
Status: Downloaded newer image for mariadb:latest
docker.io/library/mariadb:latest

$ sudo docker images -a
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
mariadb      latest    4486d64c9c3b   6 weeks ago   406MB

$ sudo docker network create --gateway 172.18.0.1 --subnet 172.18.0.0/16 maria-bridge
$ sudo docker network ls 
NETWORK ID     NAME           DRIVER    SCOPE
77d44e769d40   bridge         bridge    local
cfbfd5198082   host           host      local
8fb2418b09ad   maria-bridge   bridge    local
c64d8b941525   none           null      local
```
### 2. 마스터 데이터베이스 설정
a) 마스터 디비 컨테이너 실행  
b) 컨테이너 실행 확인  
c) 컨테이너 진입  
```bash
$ sudo docker run -d --name master-db --network maria-bridge --ip 172.18.0.2 --env MARIADB_ROOT_PASSWORD=qwe123 mariadb:latest
$ sudo docker ps -a 
CONTAINER ID   IMAGE            COMMAND                  CREATED          STATUS          PORTS      NAMES
12fcda8f5af6   mariadb:latest   "docker-entrypoint.s…"   36 minutes ago   Up 36 minutes   3306/tcp   master-db

$ sudo docker exec -it master-db /bin/bash 
```
a) 시스템 업데이트  
b) net-tools 설치  
c) vi 설치
```bash
root@12fcda8f5af6:/# apt update
root@12fcda8f5af6:/# apt install net-tools
root@12fcda8f5af6:/# apt install vim
```
a) mariadbd 설정
> binlog_format 은 3가지 옵션으로 설정이 가능하다. 아래는 각 옵션별 특징 및 장단점 
> 
> row (행 기반 로그)
> > 특징: 각 행의 변경 사항을 개별적으로 기록합니다.  
> > 장점: 데이터 일관성이 높고, 복제 시 정확한 데이터 복제가 가능합니다.  
> > 단점: 로그 크기가 커질 수 있습니다.  
> 
> statement (문 기반 로그)
> > 특징: 실행된 SQL 문을 그대로 기록합니다.  
> > 장점: 로그 크기가 작고, 저장 공간을 절약할 수 있습니다.  
> > 단점: 비결정적 함수나 복잡한 트랜잭션에서는 데이터 일관성이 떨어질 수 있습니다.
> 
> mixed (혼합 로그)
> > 특징: 기본적으로 STATEMENT 형식을 사용하지만, 특정 상황에서는 자동으로 ROW 형식으로 전환됩니다.  
> > 장점: 두 형식의 장점을 결합하여 사용합니다. 비결정적 함수나 복잡한 트랜잭션에서는 ROW 형식을 사용하여 데이터 일관성을 유지합니다.  
> > 단점: 설정이 복잡할 수 있습니다.
>  

  


```bash
root@12fcda8f5af6:/# vi /etc/mysql/mariadb.conf.d/50-server.cnf
[mariadbd]
log-bin = mysql-bin 
server-id = 1 
binlog_format = row 
```
a) 시스템 재 시작  
b) 마스터 설정 여부 확인 (show master status 커맨드를 입력했을때 아래와 같이 File, Position 확인이 가능하면 된다)
```bash
root@12fcda8f5af6:/# exit
$ sudo docker restart master-db
$ sudo docker exec -it master-db mariadb -u root -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 4
Server version: 11.4.2-MariaDB-ubu2404-log mariadb.org binary distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show master status;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000001 |      328 |              |                  |
+------------------+----------+--------------+------------------+
1 row in set (0.000 sec) 
```
### 3. 슬레이드 데이터베이스 설정
a) 슬레이브 디비 컨테이너 실행  
b) 정상 실행 여부 확인      
c) 슬레이브 디비 시스템 업데이트  
d) 슬레이브 디비 net-tools, vi 설치  
e) 슬레이브 디비 아이피 확인
```bash
$ sudo docker run -d --name slave-db1 --network maria-bridge --ip 172.18.0.3 --env MARIADB_ROOT_PASSWORD=qwe123 mariadb:latest
$ sudo docker ps -a 
CONTAINER ID   IMAGE            COMMAND                  CREATED          STATUS          PORTS      NAMES
63f195316d1f   mariadb:latest   "docker-entrypoint.s…"   18 minutes ago   Up 18 minutes   3306/tcp   slave-db1
12fcda8f5af6   mariadb:latest   "docker-entrypoint.s…"   36 minutes ago   Up 36 minutes   3306/tcp   master-db

$ sudo docker exec -it slave-db1 apt update
Hit:1 http://archive.ubuntu.com/ubuntu noble InRelease
Hit:2 http://security.ubuntu.com/ubuntu noble-security InRelease
Hit:4 http://archive.ubuntu.com/ubuntu noble-updates InRelease
Hit:5 http://archive.ubuntu.com/ubuntu noble-backports InRelease
Hit:3 https://archive.mariadb.org/mariadb-11.4.2/repo/ubuntu noble InRelease
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
2 packages can be upgraded. Run 'apt list --upgradable' to see them.

$ sudo docker exec -it slave-db1 apt install net-tools vi
$ sudo docker exec -it slave-db1 ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.18.0.3  netmask 255.255.0.0  broadcast 172.18.255.255
        ether 02:42:ac:12:00:03  txqueuelen 0  (Ethernet)
        RX packets 3292  bytes 24635832 (24.6 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2717  bytes 186770 (186.7 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
f) 슬레이브 디비 mariadbd 설정    
g) 컨테이너 재 시작   
h) 컨테이너 진입   
i) 마스터 디비 접속 체크   
```bash
$ sudo docker exec -it slave-db1 vi /etc/mysql/mariadb.conf.d/50-server.cnf
[mariadbd]
log-bin = mysql-bin 
server-id = 2 
binlog_format = row

$ sudo docker restart slave-db1
$ sudo docker exec -it slave-db1 /bin/bash

root@63f195316d1f:/# mariadb -h 172.18.0.2 -u root -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 5
Server version: 11.4.2-MariaDB-ubu2404-log mariadb.org binary distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> exit;
root@63f195316d1f:/# 
```
j) 마스터 디비 연결  
k) 슬레이브 시작  
l) 슬레이브 상태 체크 (Waiting for master to send event 라고 나온다면 정상)
```bash
root@63f195316d1f:/# mariadb -u root -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 5
Server version: 11.4.2-MariaDB-ubu2404-log mariadb.org binary distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]>
MariaDB [(none)]> CHANGE MASTER TO MASTER_HOST="172.18.0.2", MASTER_USER="root", MASTER_PASSWORD="qwe123", MASTER_PORT=3306, MASTER_LOG_FILE="mysql-bin.000001", MASTER_LOG_POS=328;
Query OK, 0 rows affected, 1 warning (0.032 sec)

MariaDB [(none)]> start slave;
Query OK, 0 rows affected (0.000 sec)

MariaDB [(none)]> show slave status;
+----------------------------------+-------------+-------------+-------------+---------------+------------------+---------------------+-------------------------+---------------+-----------------------+------------------+-------------------+-----------------+---------------------+--------------------+------------------------+-------------------------+-----------------------------+------------+------------+--------------+---------------------+-----------------+-----------------+----------------+---------------+--------------------+--------------------+--------------------+-----------------+-------------------+----------------+-----------------------+-------------------------------+---------------+---------------+----------------+----------------+-----------------------------+------------------+----------------+--------------------+------------+-------------+-------------------------+-----------------------------+---------------+-----------+---------------------+--------------------------------------------------------+------------------+--------------------------------+----------------------------+----------------------+
| Slave_IO_State                   | Master_Host | Master_User | Master_Port | Connect_Retry | Master_Log_File  | Read_Master_Log_Pos | Relay_Log_File          | Relay_Log_Pos | Relay_Master_Log_File | Slave_IO_Running | Slave_SQL_Running | Replicate_Do_DB | Replicate_Ignore_DB | Replicate_Do_Table | Replicate_Ignore_Table | Replicate_Wild_Do_Table | Replicate_Wild_Ignore_Table | Last_Errno | Last_Error | Skip_Counter | Exec_Master_Log_Pos | Relay_Log_Space | Until_Condition | Until_Log_File | Until_Log_Pos | Master_SSL_Allowed | Master_SSL_CA_File | Master_SSL_CA_Path | Master_SSL_Cert | Master_SSL_Cipher | Master_SSL_Key | Seconds_Behind_Master | Master_SSL_Verify_Server_Cert | Last_IO_Errno | Last_IO_Error | Last_SQL_Errno | Last_SQL_Error | Replicate_Ignore_Server_Ids | Master_Server_Id | Master_SSL_Crl | Master_SSL_Crlpath | Using_Gtid | Gtid_IO_Pos | Replicate_Do_Domain_Ids | Replicate_Ignore_Domain_Ids | Parallel_Mode | SQL_Delay | SQL_Remaining_Delay | Slave_SQL_Running_State                                | Slave_DDL_Groups | Slave_Non_Transactional_Groups | Slave_Transactional_Groups | Replicate_Rewrite_DB |
+----------------------------------+-------------+-------------+-------------+---------------+------------------+---------------------+-------------------------+---------------+-----------------------+------------------+-------------------+-----------------+---------------------+--------------------+------------------------+-------------------------+-----------------------------+------------+------------+--------------+---------------------+-----------------+-----------------+----------------+---------------+--------------------+--------------------+--------------------+-----------------+-------------------+----------------+-----------------------+-------------------------------+---------------+---------------+----------------+----------------+-----------------------------+------------------+----------------+--------------------+------------+-------------+-------------------------+-----------------------------+---------------+-----------+---------------------+--------------------------------------------------------+------------------+--------------------------------+----------------------------+----------------------+
| Waiting for master to send event | 172.18.0.2  | root        |        3306 |            60 | mysql-bin.000001 |                 328 | mysqld-relay-bin.000002 |           555 | mysql-bin.000001      | Yes              | Yes               |                 |                     |                    |                        |                         |                             |          0 |            |            0 |                 328 |             865 | None            |                |             0 | Yes                |                    |                    |                 |                   |                |                     0 | Yes                           |             0 |               |              0 |                |                             |                1 |                |                    | No         |             |                         |                             | optimistic    |         0 |                NULL | Slave has read all relay log; waiting for more updates |                0 |                              0 |                          0 |                      |
+----------------------------------+-------------+-------------+-------------+---------------+------------------+---------------------+-------------------------+---------------+-----------------------+------------------+-------------------+-----------------+---------------------+--------------------+------------------------+-------------------------+-----------------------------+------------+------------+--------------+---------------------+-----------------+-----------------+----------------+---------------+--------------------+--------------------+--------------------+-----------------+-------------------+----------------+-----------------------+-------------------------------+---------------+---------------+----------------+----------------+-----------------------------+------------------+----------------+--------------------+------------+-------------+-------------------------+-----------------------------+---------------+-----------+---------------------+--------------------------------------------------------+------------------+--------------------------------+----------------------------+----------------------+
1 row in set (0.000 sec)
```
### 4. 마스터 테이블 및 데이터 생성
```bash
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.000 sec)

MariaDB [(none)]> create database test1;
Query OK, 1 row affected (0.000 sec)

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| test1              |
+--------------------+
5 rows in set (0.000 sec)

MariaDB [(none)]> use test1;
Database changed
MariaDB [test1]> show tables;
Empty set (0.000 sec)

MariaDB [test1]> create table tmp (ttt varchar(100));
Query OK, 0 rows affected (0.024 sec)

MariaDB [test1]> show tables;
+-----------------+
| Tables_in_test1 |
+-----------------+
| tmp             |
+-----------------+
1 row in set (0.000 sec)

MariaDB [test1]> insert into tmp (ttt) values ('1111');
Query OK, 1 row affected (0.002 sec)

MariaDB [test1]> select * from tmp;
+------+
| ttt  |
+------+
| 1111 |
+------+
1 row in set (0.000 sec)
```
### 5. 슬레이브 복제 여부 체크 
마스터 디비에서 데이터베이스 및 테이블을 생성하고 데이터를 Insert 했으니, 슬레이브 디비에서 동일 내용이 보이는지 확인
```bash
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| test1              |
+--------------------+
5 rows in set (0.000 sec)

MariaDB [(none)]> use test1;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [test1]> show tables;
+-----------------+
| Tables_in_test1 |
+-----------------+
| tmp             |
+-----------------+
1 row in set (0.000 sec)

MariaDB [test1]> select * from tmp;
+------+
| ttt  |
+------+
| 1111 |
+------+
1 row in set (0.001 sec)
```
잘 작동한다. :)