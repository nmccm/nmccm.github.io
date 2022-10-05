---
title: "(리눅스) 특정 디렉토리 (하위 디렉토리 포함) 확장자, 확장자 수, 용량 체크 쉘 스크립트"
categories: 
- Linux 
- CentOS
- Ubuntu
- Shell
- Script
tags:
- Linux 
- CentOS
- Ubuntu
- Shell
- Script
---

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

```bash
$ sudo chmod +x z_ext.sh
$ ./z_ext.sh > z_result.txt
```

