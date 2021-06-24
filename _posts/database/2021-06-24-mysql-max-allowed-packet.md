---
title: "(MySQL, MariaDB) (2013) Lost connection to MySQL server during query"
date: 2021-06-24 16:49:28 -0400
categories: mysql
tag: MySQL MariaDB
---

MySQL 또는 MariaDB 에 dump 된 데이터를 import 또는 export 시킬때 볼수 있는 에러,
max_allowd_packet size 를 늘려줌으로써 해결가능

아래 명령은 해당 세션의 max_allowd_packet 을 1G 로 늘려준다.

```sql
SET session max_allowed_packet = 1024 * 1024 * 1600;
SHOW VARIABLES LIKE '%packet';
```

아래 명령은 데이터베이스 서버의 max_allowd_packet 을 1G 로 늘려준다.

```sql
SET GLOBAL max_allowed_packet = 1024 * 1024 * 1600
SHOW GLOBAL VARIABLES LIKE '%packet';;
```
