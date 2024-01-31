---
title: "NodeJS 로 윈도우 실행파일(exe) 만들기"
categories:
- NodeJS
tags:
- NodeJS
---

1. 개발환경  

```bash
OS : Ubuntu 20.04 
NPM : 6.14.4
NodeJS : v10.19.0
```
2. NodeJS 설치

시스템에 NodeJS 설치 여부를 체크하여 아래와 같이 설치된 버전이 없다면 설치를 진행

```bash
$ node --version
Command 'nodejs' not found, but can be installed with:
sudo apt install nodejs

$ sudo apt install nodejs
Reading package lists... Done
Building dependency tree
Reading state information... Done
E: Unable to locate package nodejs

$ sudo apt update
Hit:1 http://archive.ubuntu.com/ubuntu focal InRelease
Get:2 http://security.ubuntu.com/ubuntu focal-security InRelease [114 kB]
Get:3 http://archive.ubuntu.com/ubuntu focal-updates InRelease [114 kB]
.
.
.
.
Get:42 http://archive.ubuntu.com/ubuntu focal-backports/multiverse amd64 c-n-f Metadata [116 B]
Fetched 29.9 MB in 5s (5440 kB/s)
Reading package lists... Done
Building dependency tree
Reading state information... Done
127 packages can be upgraded. Run 'apt list --upgradable' to see them.

$ sudo apt install nodejs
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following additional packages will be installed:
  libc-ares2 libnode64 nodejs-doc
Suggested packages:
  npm
The following NEW packages will be installed:
  libc-ares2 libnode64 nodejs nodejs-doc
0 upgraded, 4 newly installed, 0 to remove and 127 not upgraded.
Need to get 6807 kB of archives.
After this operation, 30.7 MB of additional disk space will be used.
Do you want to continue? [Y/n] you

Get:1 http://archive.ubuntu.com/ubuntu focal-updates/main amd64 libc-ares2 amd64 1.15.0-1ubuntu0.4 [36.9 kB]Get:2 http://archive.ubuntu.com/ubuntu focal-updates/universe amd64 libnode64 amd64 10.19.0~dfsg-3ubuntu1.3 [5766 kB]
Get:3 http://archive.ubuntu.com/ubuntu focal-updates/universe amd64 nodejs-doc all 10.19.0~dfsg-3ubuntu1.3 [943 kB]
.
.
.
Setting up nodejs (10.19.0~dfsg-3ubuntu1.3) ...
update-alternatives: using /usr/bin/nodejs to provide /usr/bin/js (js) in auto mode
Processing triggers for libc-bin (2.31-0ubuntu9.9) ...
Processing triggers for man-db (2.9.1-1) ...

$ node --version
v10.19.0
```
3. NPM 설치

본인이 개발한 NodeJS 파일을 .EXE 파일로 생성하려면 pkg 필요, pkg 설치는 nodejs 패키지 관리자(npm)로 패키지 설치를 진행해야 하므로 npm 설치를 진행

```bash
$ npm --version
Command 'npm' not found, but can be installed with:
sudo apt install npm

$ sudo apt install npm
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following additional packages will be installed:
.
.
.
Get:1 http://archive.ubuntu.com/ubuntu focal-updates/universe amd64 libpython2.7-minimal amd64 2.7.18-1~20.04.3 [336 kB]
Get:2 http://archive.ubuntu.com/ubuntu focal-updates/universe amd64 python2.7-minimal amd64 2.7.18-1~20.04.3 [1280 kB]
Get:3 http://archive.ubuntu.com/ubuntu focal/universe amd64 python2-minimal amd64 2.7.17-2ubuntu4 [27.5 kB]
.
.
.
Selecting previously unselected package libipc-system-simple-perl.
Preparing to unpack .../045-libipc-system-simple-perl_1.26-1_all.deb ...
Unpacking libipc-system-simple-perl (1.26-1) ...
Selecting previously unselected package libfile-basedir-perl.
Preparing to unpack .../046-libfile-basedir-perl_0.08-1_all.deb ...
Unpacking libfile-basedir-perl (0.08-1) ...
.
.
.
Setting up node-slide (1.1.6-2) ...
Setting up node-delayed-stream (1.0.0-4) ...
Setting up node-isstream (0.1.2+dfsg-1) ...
Setting up node-builtins (1.0.3-1) ...
.
.
.
Setting up libxml-parser-perl (2.46-1) ...
Setting up libxml-twig-perl (1:3.50-2) ...
Setting up libnet-dbus-perl (1.2.0-1) ...
Processing triggers for man-db (2.9.1-1) ...
Processing triggers for mime-support (3.64ubuntu1) ...
Processing triggers for libc-bin (2.31-0ubuntu9.9) ...

$ npm --version
6.14.4
```

4. 패키지(pkg) 설치

```bash
$ sudo npm install -g pkg
/usr/local/bin/pkg -> /usr/local/lib/node_modules/pkg/lib-es5/bin.js
+ pkg@5.8.1
added 127 packages from 94 contributors in 8.243s
```

5. 개발 (프로젝트 폴더 생성)

```bash
$ cd /home/devteam
$ mkdir nodejs_exe
$ cd nodejs_exe
$ mkdir src
$ mkdir bin
$ ls -al
total 16
drwxr-xr-x 4 devteam devteam 4096 Jan 31 14:05 .
drwxr-xr-x 8 devteam devteam 4096 Jan 31 14:06 ..
drwxr-xr-x 2 devteam devteam 4096 Jan 31 14:06 bin
drwxr-xr-x 2 devteam devteam 4096 Jan 31 14:05 src
```

5. 개발 (index.js)

```bash
$ vi ./src/index.js

console.log('success');
```

6. 빌드

```bash
$ cd /home/devteam/nodejs_exe/bin
$ pkg ../src/index.js --targets node10-win-x64
```

디렉토리 내용을 살펴보면 (ls -al) index.exe 파일이 생성된것을 확인할 수 있다. 해당 파일을 윈도우로 옮겨 실행해보면 정상적으로 success 메세지를 확인할 수 있다. 
