---
title: "라라벨 엘로퀀트 GroupBy (Laravel Eloquent)"
categories: 
- Laravel 
- Eloquent
tags:
- Laravel
- Eloquent
---

- Ansi SQL

```sql
select 
    c_date, count(*) as cnt    
from tables 
where c_date between '2021-06-01' and '2021-06-30'
group by c_date
```
- eloquent

```php
$results = TableModel::query()
    ->select('c_date', DB::raw('count(*) as cnt'))
    ->whereBetWeen('c_date', [
        Carbon::now()->addDays(-30)->toDateString(),
        Carbon::now()->addDays(-1)->toDateString()
    ])
    ->groupBy('c_date')
    ->get();
```
