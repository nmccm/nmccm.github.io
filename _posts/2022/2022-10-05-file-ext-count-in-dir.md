---
title: "(리눅스) 특정 디렉토리 (하위 디렉토리 포함) 확장자, 확장자 수, 용량 체크 쉘 스크립트"
categories: 
- Linux
tags:
- CentOS
- Ubuntu
---

서버를 운용하다보면 여러 케이스로 Find 명령어를 자주 사용하게 된다. 이번에는 특정 폴더 하위에 여러 폴더들이 꽤 깊게 폴더링 되어 있는 상황에서  한번의 명령어로 해당 폴더(하위 폴더 포함)내에 있는 파일들의 확장자 종류, 그 종류의 파일 갯수, 그 종류의 파일들이 가지고 있는 용량을 한번에 확인 가능한 명령어를 구글링하여 얻게되어 여기에 기록한다.

### 1. 환경

```bash
OS : Ubuntu 18.04
```

### 2. 스크립트 작성

리눅스(linux) find 기반 특정 디렉토리 (하위 디렉토리 포함) 확장자, 확장자 수, 용량 체크 쉘 스크립트

```bash
$ vi z_ext.sh
```

```bash
#!/bin/sh
for Dir in `find . -type d`
do
   echo "- $Dir "
   for Ext in `find $Dir -type f -maxdepth 1| awk -F/ '{printf "%s\n",$NF}' | grep "\." | sed -e 's/.*\.\([a-zA-Z0-9].*\)/\1/g'  | sort -u`
   do
      Count=`find $Dir -type f -maxdepth 1| awk -F/ '{printf "%s\n",$NF}' | grep -c "\.${Ext}$"`
      Size=`find $Dir -name "*.$Ext" -maxdepth 1 -ls | awk 'BEGIN{sum=0} {sum=sum+$7} END{print sum}'`
      echo $Dir $Ext $Count $Size
   done | sort -rn
done
```

### 3. 실행

```bash
$ sudo chmod +x z_ext.sh
$ ./z_ext.sh > z_result.txt
```

