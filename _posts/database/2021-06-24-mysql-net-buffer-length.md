---
title: "(MySQL, MariaDB) (1153): Got a packet bigger than 'max_allowed_packet' bytes"
date: 2021-06-24 17:58:28 -0400
categories: MySQL
tag: 
  - MySQL
  - MariaDB
  - Database
---

MySQL 또는 MariaDB 에 dump 된 데이터를 import 또는 export 시킬때 볼수 있는 에러,
net_buffer_length size 를 늘려줌으로써 해결가능

아래 명령은 해당 세션의 net_buffer_length 늘려준다.

```sql
SET session net_buffer_length = 1000000;
SHOW VARIABLES LIKE 'net_buffer_length';
```

아래 명령은 데이터베이스 서버의 net_buffer_length 를 늘려준다.

```sql
SET GLOBAL net_buffer_length = 1000000;
SHOW GLOBAL VARIABLES LIKE 'net_buffer_length';;
```
