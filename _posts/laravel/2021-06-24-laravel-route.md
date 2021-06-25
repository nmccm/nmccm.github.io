---
title: "라라벨 라우트 (Laravel Route)"
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
