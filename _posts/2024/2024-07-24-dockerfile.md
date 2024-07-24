---
title: "Dockerfile 로 Mariadb 도커 이미지 생성 방법"
categories: 
- Docker
- MariaDB
tags:
- Docker
- MariaDB
---

도커는 2015년 이래 리눅스 컨테이너 기술 부분에서 사실상 업계 표준이 되어 가고 있다. 그렇기에 가장 큰 장점으로는 사실상 업계 표준이니만큼 사용자들이 작성해둔 소프트웨어 패키지/이미지들이 넘쳐나고 있어서 접근성과 사용하기 좋다는 장점이 있으며 최근에는 클라우드 컴퓨팅에 대해 교육을 진행하는 여러 교육기관에서도 도커에 대한 커리큘럼을 추가하는 경우가 많아지고 있다.(출처 : 나무위키) 

### 0. 환경

```bash
a) Window Hyper-V 10.0.22621.1
b) Ubuntu 24.04 LTS
```
 
### 1. 기초설정
net-tools 패키지는 아이피 및 네트워크 정보를 확인 가능하도록 해주는 ifconfig 명령어를 포함하고 있고, openssh-server 는 putty 와 같은 프로그램으로 하이퍼바이저 위에 설치된 ubuntu 시스템에 ssh 접속이 가능하도록 해준다. (이것이 가능하려면 방화벽 설정을 해줘야 하는데, 테스트 서버이고 내부 IP 에서만 사용될 시스템이므로 일단 방화벽은 off)
```bash
$ sudo apt update
$ sudo apt install net-tools
$ sudo apt install openssh-server
$ sudo ufw disable
$ sudo ufw status
```

### 2. Docker 설치 및 설정
아래 명령어는 도커 설치를 하고, 제대로 설치가 되어 있는지 확인. 설치된 도커 서비스를 실행해주고, 재부팅시에도 도커 서비스가 유지되도록 시작 프로그램에 등록하는 과정이다. 마지막으로 도커 서비스가 제대로 실행되고 있는지 프로세스 확인도 잊지 말자
```bash
$ sudo install docker.io
install: missing destination file operand after 'docker.io'
Try 'install --help' for more information.
devteampick@lmclocal:~$ sudo apt install docker.io
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  bridge-utils containerd dns-root-data dnsmasq-base pigz runc ubuntu-fan
Suggested packages:
  ifupdown aufs-tools cgroupfs-mount | cgroup-lite debootstrap docker-buildx docker-compose-v2 docker-doc rinse zfs-fuse | zfsutils
The following NEW packages will be installed:
  bridge-utils containerd dns-root-data dnsmasq-base docker.io pigz runc ubuntu-fan
0 upgraded, 8 newly installed, 0 to remove and 41 not upgraded.
Need to get 76.8 MB of archives.
After this operation, 289 MB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 http://kr.archive.ubuntu.com/ubuntu noble/universe amd64 pigz amd64 2.8-1 [65.6 kB]
Get:2 http://kr.archive.ubuntu.com/ubuntu noble/main amd64 bridge-utils amd64 1.7.1-1ubuntu2 [33.9 kB]
Get:3 http://kr.archive.ubuntu.com/ubuntu noble/main amd64 runc amd64 1.1.12-0ubuntu3 [8,599 kB]
.
.
.
Processing triggers for man-db (2.12.0-4build2) ...
Scanning processes...
Scanning linux images...
Running kernel seems to be up-to-date.
No services need to be restarted.
No containers need to be restarted.
No user sessions are running outdated binaries.
No VM guests are running outdated hypervisor (qemu) binaries on this host.

$ dpkg -l | grep docker
ii  docker.io                            24.0.7-0ubuntu4                         amd64        Linux container runtime

$ sudo systemctl start docker
$ sudo systemctl enable docker

$ ps -ef | grep docker
root        3149       1  0 06:25 ?        00:00:00 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
devteam+    3339    2409  0 06:28 pts/0    00:00:00 grep --color=auto docker
```

### 3. 프로젝트 폴더 생성 및 도커파일 작성
우분투 최신 버전을 기준으로 마리아 디비 도커 이미지를 생성할 계획이므로 우선 우분트 이미지를 다운로드 받는다. 프로젝트 폴더를 생성하고, 해당 폴더에 Dockerfile 을 작성한다.
```bash
$ cd ~
$ mkdir docker_mariadb
$ cd docker_mariadb/

$ sudo docker pull ubuntu:latest
latest: Pulling from library/ubuntu
9c704ecd0c69: Already exists
Digest: sha256:2e863c44b718727c860746568e1d54afd13b2fa71b160f5cd9058fc436217b30
Status: Downloaded newer image for ubuntu:latest
docker.io/library/ubuntu:latest

$ sudo docker images -a
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
ubuntu       latest    35a88802559d   6 weeks ago   78.1MB

$ vi Dockerfile
FROM ubuntu

RUN apt update && apt install -y mariadb-server mariadb-client

EXPOSE 3306

CMD ["mysqld_safe"]
```

### 4. 빌드

```bash
$ sudo docker build --force-rm -t lmc-mariadb:latest .
DEPRECATED: The legacy builder is deprecated and will be removed in a future release.
            Install the buildx component to build images with BuildKit:
            https://docs.docker.com/go/buildx/

Sending build context to Docker daemon  2.048kB
Step 1/4 : FROM ubuntu
 ---> 35a88802559d
Step 2/4 : RUN apt update && apt install -y mariadb-server mariadb-client
 ---> Running in b251f0a7a82c

WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

Get:1 http://archive.ubuntu.com/ubuntu noble InRelease [256 kB]
.
.
.
invoke-rc.d: could not determine current runlevel
invoke-rc.d: policy-rc.d denied execution of restart.
Removing intermediate container b251f0a7a82c
 ---> 5e9277620019
Step 3/4 : EXPOSE 3306
 ---> Running in 92815a667102
Removing intermediate container 92815a667102
 ---> 8b9514eca21b
Step 4/4 : CMD ["mysqld_safe"]
 ---> Running in 20fb525cba8f
Removing intermediate container 20fb525cba8f
 ---> 48b93c78ab4a
Successfully built 48b93c78ab4a
Successfully tagged lmc-mariadb:latest
```

### 5. 이미지 확인
정상적으로 이미지가 생성되었다. 다만 메세지에서도 보이듯이 더이상 build 명령어를 통한 빌드는 곧 릴리즈에서 제거되어 제 기능을 하지 않아, 빌드 중간중간 생성되는 임시 이미지들이 제거되지(--force-rm) 않은 모습을 볼 수 있다.
```bash
$ sudo docker images -a
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
<none>        <none>    5e9277620019   2 minutes ago   490MB
<none>        <none>    8b9514eca21b   2 minutes ago   490MB
lmc-mariadb   latest    48b93c78ab4a   2 minutes ago   490MB
ubuntu        latest    35a88802559d   6 weeks ago     78.1MB
```
BuildKit 으로 재 빌드 위해 생성된 이미지 모두 제거 (메인 이미지를 삭제하면 임시 이미지도 같이 삭제 됨) 
```bash
$ sudo docker images -a
REPOSITORY    TAG       IMAGE ID       CREATED          SIZE
<none>        <none>    5e9277620019   13 minutes ago   490MB
<none>        <none>    8b9514eca21b   13 minutes ago   490MB
lmc-mariadb   latest    48b93c78ab4a   13 minutes ago   490MB
ubuntu        latest    35a88802559d   6 weeks ago      78.1MB

$ sudo docker rmi lmc-mariadb
Untagged: lmc-mariadb:latest
Deleted: sha256:48b93c78ab4a9cdc566d53d4b97b0578366ad2ced673a4d44a424b0690717f55
Deleted: sha256:8b9514eca21b5ab5f58620cacb925346814b6fbd8c5b513da7d06ac46c473c2f
Deleted: sha256:5e9277620019b681d930581277b2ccce19a67c1b4edaf31a8b694bb29116bf8b
Deleted: sha256:c65556c7c8cff3251260b2e015b1764ab9adb7450a615deac62eaf514177845c

$ sudo docker images -a
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
ubuntu       latest    35a88802559d   6 weeks ago   78.1MB
```

### 6. BuildKit 설치 및 빌드

```bash
$ sudo apt install docker-buildx
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  docker-buildx
0 upgraded, 1 newly installed, 0 to remove and 41 not upgraded.
Need to get 12.7 MB of archives.
After this operation, 53.8 MB of additional disk space will be used.
Get:1 http://kr.archive.ubuntu.com/ubuntu noble-updates/universe amd64 docker-buildx amd64 0.12.1-0ubuntu2.1 [12.7 MB]
Fetched 12.7 MB in 3s (3,811 kB/s)
Selecting previously unselected package docker-buildx.
(Reading database ... 84082 files and directories currently installed.)
Preparing to unpack .../docker-buildx_0.12.1-0ubuntu2.1_amd64.deb ...
Unpacking docker-buildx (0.12.1-0ubuntu2.1) ...
Setting up docker-buildx (0.12.1-0ubuntu2.1) ...
Scanning processes...
Scanning linux images...
Running kernel seems to be up-to-date.
No services need to be restarted.
No containers need to be restarted.
No user sessions are running outdated binaries.
No VM guests are running outdated hypervisor (qemu) binaries on this host.

$ sudo docker buildx version
github.com/docker/buildx 0.12.1 0.12.1-0ubuntu2.1

$ sudo docker buildx build --platform linux/amd64 -t lmc-mariadb:latest .
[+] Building 25.4s (6/6) FINISHED                                                                                                                  docker:default
 => [internal] load .dockerignore                                                                                                                            0.1s
 => => transferring context: 2B                                                                                                                              0.0s
 => [internal] load build definition from Dockerfile                                                                                                         0.1s
 => => transferring dockerfile: 147B                                                                                                                         0.0s
 => [internal] load metadata for docker.io/library/ubuntu:latest                                                                                             0.0s
 => [1/2] FROM docker.io/library/ubuntu                                                                                                                      0.0s
 => [2/2] RUN apt update && apt install -y mariadb-server mariadb-client                                                                                    24.1s
 => exporting to image                                                                                                                                       1.0s
 => => exporting layers                                                                                                                                      1.0s
 => => writing image sha256:0ea9cd6379c4035cbcdb74c4b784591ff9ac36c269171134f7a6159e77175edd                                                                 0.0s
 => => naming to docker.io/library/lmc-mariadb:latest
 
$ sudo docker images -a
REPOSITORY    TAG       IMAGE ID       CREATED              SIZE
lmc-mariadb   latest    0ea9cd6379c4   About a minute ago   490MB
ubuntu        latest    35a88802559d   6 weeks ago          78.1MB             
```

### 7. 이미지 실행 (컨테이너)
```bash
$ sudo docker images -a
REPOSITORY    TAG       IMAGE ID       CREATED              SIZE
lmc-mariadb   latest    0ea9cd6379c4   About a minute ago   490MB
ubuntu        latest    35a88802559d   6 weeks ago          78.1MB

$ sudo docker run -d --name lmc-mariadb --env MARIADB_ROOT_PASSWORD=abcd1234  lmc-mariadb:latest
c824055a661088dbb5dddaff586b769243cc11b058861b7aef4dc47096ef87b6

$ sudo docker ps -a
CONTAINER ID   IMAGE                COMMAND         CREATED         STATUS         PORTS      NAMES
c824055a6610   lmc-mariadb:latest   "mysqld_safe"   4 seconds ago   Up 3 seconds   3306/tcp   lmc-mariadb

$ sudo docker exec -it lmc-mariadb mariadb -u root -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 3
Server version: 10.11.8-MariaDB-0ubuntu0.24.04.1 Ubuntu 24.04

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.009 sec)

MariaDB [(none)]>
```

잘 되는것 같습니다. :)