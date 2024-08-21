---
title: "CentOS 에서 Docker 설치 및 설정"
categories: 
- Docker
tags:
- Docker
---
Docker는 애플리케이션을 컨테이너라는 표준화된 유닛으로 패키징하여 신속하게 구축, 테스트 및 배포할 수 있는 소프트웨어 플랫폼입니다. 컨테이너에는 애플리케이션 실행에 필요한 모든 요소가 포함되어 있어, 환경에 구애받지 않고 일관된 실행을 보장합니다. Docker는 리눅스 컨테이너 기술을 기반으로 하며, 가상 머신보다 가볍고 빠르게 프로세스를 격리할 수 있습니다. 이를 통해 개발 환경 구축뿐만 아니라 서버 운영에서도 많은 인기를 얻고 있습니다. (출처 : Window AI Copilot)

### 0. 환경
> Window Hyper-V 10.0.22621.1  
> Ubuntu 24.04 LTS

### 1. Docker 설치
루트권한으로 진행
```bash
# yum install -y yum-utils device-mapper-persistent-data lvm2
# yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
# yum install docker-ce docker-ce-cli containerd.io
# systemctl start docker
# systemctl enable docker

# ps -ef | grep docker
root         878       1  0 01:07 ?        00:00:01 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
devteam+    3170    3160  0 01:43 pts/2    00:00:00 grep --color=auto docker
```
