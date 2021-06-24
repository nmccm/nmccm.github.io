---
title: "라라벨 엘로퀀트 컬럼 서브쿼리 (Laravel Eloquent)"
date: 2021-06-24 16:08:28 -0400
categories: laravel eloquent
---

- Ansi SQL

```sql
select 
    a.*,
    (select name from B where B.id = A.id) as bName
from A as a
where a.id = 10
```
- eloquent

```php
$query = A::query()
    ->select(
        'A.*',
        DB::raw('(select name from B where B.id = A.id) as bName')
    )
    ->where('A.id', 10)
    ->get()
```
