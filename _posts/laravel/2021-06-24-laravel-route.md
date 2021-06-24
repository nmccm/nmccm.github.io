---
title: "라라벨 라우트 (Laravel Route)"
date: 2021-06-24 14:07:28 -0400
categories: Laravel
tags:
    - Laravel
    - Route
---

- 라우트 정의

```php
// routes/web.php
Route::get('/', function() {
    return 'Hello, world';
});

Route::get('/about', function() {
    return view('about', [
        'data' => $data,
    ]);
});
```
