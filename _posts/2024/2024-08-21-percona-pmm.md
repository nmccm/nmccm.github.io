---
title: "PMM (in CentOS Docker)"
categories: 
- PMM 
tags:
- PMM
---
Percona Monitoring and Management (PMM)은 MySQL, PostgreSQL, MongoDB와 같은 데이터베이스를 위한 오픈 소스 모니터링, 관찰성, 관리 도구입니다. PMM을 사용하면 모든 데이터베이스의 성능 메트릭을 한 곳에서 확인할 수 있습니다.
### 1. 환경
- Window Hyper-V 10.0.22621.1  
- Ubuntu 24.04 LTS
- Docker 18.09.8

### 2. Docker 서비스 확인

```bash
# ps -ef | grep docker
root         878       1  0 01:07 ?        00:00:01 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
devteam+    3170    3160  0 01:43 pts/2    00:00:00 grep --color=auto docker
```

### 3. PMM Server 설치

```bash
# docker pull percona/pmm-server
# docker create --name pmm-data percona/pmm-server /bin/true
# docker run --detach --restart always --publish 8080:80 --publish 4433:443 --volumes-from pmm-data --name pmm-server percona/pmm-server
```

### 4. PMM Client 설치

```bash
# yum install -y https://repo.percona.com/yum/percona-release-latest.noarch.rpm
# yum install -y pmm2-client
```

### 5. Database 계정 설정

```bash
MariaDB [(none)]> create user pmm@'%' identified by '123456' WITH MAX_USER_CONNECTIONS 10;
MariaDB [(none)]> GRANT ALL PRIVILEGES ON *.* TO 'pmm'@'%' WITH GRANT OPTION;
MariaDB [(none)]> GRANT SELECT, PROCESS, SUPER, REPLICATION CLIENT, RELOAD, show view ON *.* TO 'pmm'@'%' ;
MariaDB [(none)]> GRANT SELECT, UPDATE, DELETE, DROP ON performance_schema.* TO 'pmm'@'%';
MariaDB [(none)]> FLUSH PRIVILEGES;

MariaDB [(none)]> create user pmm@'localhost' identified by '123456' WITH MAX_USER_CONNECTIONS 10;
MariaDB [(none)]> GRANT ALL PRIVILEGES ON *.* TO 'pmm'@'localhost' WITH GRANT OPTION;
MariaDB [(none)]> GRANT SELECT, PROCESS, SUPER, REPLICATION CLIENT, RELOAD, show view ON *.* TO 'pmm'@'localhost';
MariaDB [(none)]> GRANT SELECT, UPDATE, DELETE, DROP ON performance_schema.* TO 'pmm'@'localhost';
MariaDB [(none)]> FLUSH PRIVILEGES;
```

### 6. PMM 서버 연결

```bash
# pmm-admin config --server-insecure-tls --server-url=https://admin:admin@111.111.111.100:4433
# pmm-admin add mysql --query-source=perfschema --username=pmm --password='123456'
```

### 7. PMM 연결 정보 확인

```bash
# pmm-admin list --server-url=https://admin:admin@111.111.111.100:4433
```

### 8. Agent 실행 여부 확인 및 중지

```bash
# systemctl stop pmm-agent
# systemctl status pmm-agent
# systemctl start pmm-agent
```