---
title: "Laravel 8 에서 Echo 시스템 구축기"
categories: 
- PHP
- Laravel
tags:
- Laravel8
- Laravel-echo-server
---

라라벨(Laravel)의 브로드캐스팅(Broadcasting)은 실시간 이벤트를 처리하고 클라이언트에게 푸시하는 기능을 제공합니다. 이것은 웹 소켓(WebSocket)을 기반으로 하거나, 추상화된 Redis, Pusher 등의 서비스를 통해 구현될 수 있습니다. 여기에서는 라라벨의 브로드캐스팅이 Redis를 사용하는 경우를 중심으로 설명하겠습니다.

### 1. 개발환경  

```bash
OS : Window 10 pro
XAMPP
- Apache : 2.4
- PHP : 7.3.27
- MariaDB 10.4.18
- Laravel 8.49
```

### 2. 라라벨 프로젝트 생성 및 설정

새로운 라라벨 프로젝트를 생성하고, 필요한 패키지들을 설치한다.

```bash
$ composer create-project --prefer-dist laravel/laravel echo2
$ cd echo2
$ composer require predis/predis
$ npm install -g laravel-echo-server
```
### 3. 필요 파일 설정

브로드캐스팅 핸들러(이벤트가 발생하면 변경된 데이터 보관 및 가공하는 CLASS) 생성. 아래 커맨드는 \App\Event\SiteMessage 클래스를 생성한다. 

```bash
php artisan make:event SiteMessage
```

현재 Laravel 8.49 에서는 SiteMessage 클래스를 artisan 으로 생성시에 ShouldBroadcast 가 Implement 되지 않으므로 SiteMessage 클래스 정의 부분을 아래처럼 변경한다.

```php
class SiteMessage implements ShouldBroadcast
```

broadcastOn 메쏘드의 리턴값을 PrivateChannel 에서 Channel 로 변경하고, 통신 채널명을 site-message 로 변경

```php
public function broadcastOn() {
    return new Channel('site-message');
}

public function broadcastWith() {
    return ['id' => 3];
}
```

.env 파일에서 다음 항목이 없으면 추가하고, 있다면 변경

```php
BROADCAST_DRIVER=redis

REDIS_HOST=127.0.0.1    // 래디스가 설치되어 있는 플랫폼의 IP 
REDIS_PASSWORD=null
REDIS_PORT=6379         // 래디스 기본 포트

LARAVEL_ECHO_PORT=6001  // 라라벨 에코 서버 기본 포트
```

config/database.php 에서 redis 관련 내용 변경

```php
    'redis' => [
        'client' => env('REDIS_CLIENT', 'predis'),
        'options' => [
            'cluster' => env('REDIS_CLUSTER', 'redis'),            
        ],
        'default' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_DB', '0'),
        ],
        'cache' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_CACHE_DB', '1'),
        ],
    ],
```

### 4. 라라벨 에코서버 환경 설정

```bash 
$ laravel-echo-server init

? Do you want to run this server in development mode? Yes
? Which port would you like to serve from? 6001
? Which database would you like to use to store presence channel members? redis
? Enter the host of your Laravel authentication server. http://localhost:8003
? Will you be serving on http or https? http
? Do you want to generate a client ID/Key for HTTP API? Yes
? Do you want to setup cross domain access to the API? Yes
? Specify the URI that may access the API: http://localhost:8003
? Enter the HTTP methods that are allowed for CORS: GET, POST
? Enter the HTTP headers that are allowed for CORS: Origin, Content-Type, X-Auth
? What do you want this config to be saved as? laravel-echo-server.json
appId: 990b044d737e7f48
key: 6445f3f75715ac09b948a01d4dcd7959
Configuration file saved. Run laravel-echo-server start to run server.
```

### 5. Node.js 관련 모듈 설치

```bash
$ npm install
$ npm install laravel-echo
$ npm install socket.io-client
```

/resources/js/laravel-echo.js 파일을 추가한다.

```js
import Echo from 'laravel-echo';
//window.io = require('socket.io-client');
window.Echo = new Echo({
    broadcaster: 'socket.io',
    host: window.location.hostname + ":6001"
});
```

webpack.mix.js 에 laravel-echo.js 파일을 추가

```js
mix.js('resources/js/app.js', 'public/js').postCss('resources/css/app.css', 'public/css', [
    //
]);
mix.js('resources/js/laravel-echo.js', 'public/js');
```

npm run dev 실행하여 public 위치로 배포한다.

```bash
$ npm run dev
```

config/app.php 에서 BroadcastServiceProvider 활성화 시킨다. (주석해제)

```php
/*
* Application Service Providers...
*/
App\Providers\AppServiceProvider::class,
App\Providers\AuthServiceProvider::class,
App\Providers\BroadcastServiceProvider::class,  // this
App\Providers\EventServiceProvider::class,
App\Providers\RouteServiceProvider::class,
```

routes/web.php 에 route 추가

```php
Route::get('/send', function () {
    event(new \App\Events\SiteMessage());
    return 'send';
});
```

/resources/views/welcome.blade.php 파일에 방금 빌드한 app.js, app.css, laravel-echo.js 파일을 포함하고, csrf-token 을 추가와 에코서버 리스너를 설정


```html
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
    <head>
        <meta name="csrf-token" content="{{ csrf_token() }}">
        <link href="{{ asset('css/app.css') }}" rel="stylesheet">
        <script src="{{ url('/js/app.js') }}" type="text/javascript"></script>
        <script src="//{{ Request::getHost() }}:{{ env('LARAVEL_ECHO_PORT') }}/socket.io/socket.io.js"></script>
        <script src="{{ url('/js/laravel-echo.js') }}" type="text/javascript"></script>
        <script>
            window.laravel_echo_port='{{ env("LARAVEL_ECHO_PORT") }}';
            window.Echo.channel('site-message').listen(
                'SiteMessage', 
                (data) => {
                    console.log(data);
                }
            );
        </script>
    </head>
    <body class="antialiased">
        welcome laravel
    </body>
</html>
```

### 6. 테스트 

라라벨 에코서버를 실행

```bash 
$ laravel-echo-server start

L A R A V E L  E C H O  S E R V E R

version 1.6.2

??Starting server in DEV mode...

?? Running at localhost on port 6001
?? Channels are ready.
?? Listening for http events...
?? Listening for redis events...

Server ready!
```

http://localhost:8003 으로 접속해보면 아래와 같은 메세지를 볼수 있다.

```bash
[?ㅼ쟾 11:33:48] - Dy7uOh4nIZoq6Pt2AAAA joined channel: site-message
```

