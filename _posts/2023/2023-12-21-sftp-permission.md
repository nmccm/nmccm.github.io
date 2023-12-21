---
title: "SFTP 상위 폴더 접근 권한 제한"
categories:
- Linux
tags:
- CentOS
- SSHD
---

SFTP 연결을 위해 계정을 생성하고 접속해보면 root 권한을 가지지 않은 계정 또한 상위 폴더를 자유롭게 이동이 가능한 부분이 확인 가능하다. 해당 이슈를 해결하기 위해 chroot(Change Root Directory) 이용하여 사용자 루트 디렉토리 설정하는 방법에 대해 CentOS 7 기준으로 작성한다.

SFTP 연결에 사용할 계정은 content 이므로 우선 계정을 생성한다. 

```linux
# useradd content
# passwd content
Changing password for user content.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
```

vi 이용하여 /etc/ssh/sshd_config 파일을 아래와 같이 편집한다. 

```linux
#Subsystem       sftp    /usr/libexec/openssh/sftp-server -f local2 -l INFO
Subsystem       sftp    internal-sftp

# Example of overriding settings on a per-user basis
Match User content
        ChrootDirectory /home/content
        X11Forwarding no
        AllowTcpForwarding no
        PermitTTY no
        ForceCommand internal-sftp
```

설정 내용 반영을 위해 sshd 서비스 재 시작

```linux
# service sshd restart
```

winscp 같은 sftp 프로토콜 지원하는 ftp 클라이언트로 content 계정 접속하면 상위 폴더로 이동할 수 없음을 확인