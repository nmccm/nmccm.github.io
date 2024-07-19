---
title: "라이브러리(web-push-php) 사용하여 웹 푸쉬 구현하기"
categories: 
- PHP
- Laravel
tags:
- Laravel8
---

웹푸시(Web Push Notification)는 PC, 태블릿, 모바일 기기에서 웹 브라우저를 통해 푸시를 받을 수 있는 마케팅 채널 중 하나입니다. 푸시라고 하면 보통 스마트폰을 통해 알림을 받는 앱푸시부터 떠오르기 마련인데요. 웹푸시도 안드로이드 모바일 폰에서 앱푸시와 동일한 형태로 푸시 메시지를 받을 수 있습니다. 

### 1. 개발환경  

```bash
- Apache : 2.4
- PHP : 7.3.27
- MariaDB 10.4.18
- Laravel 8.49
```

### 2. web-push-php 라이브러리 설치

```bash
$ composer require minishlink/web-push
```

### 3. Public & Private Key 생성

```php
namespace App\Http\Controllers;

use Minishlink\WebPush\WebPush;
use Minishlink\WebPush\Subscription;
use Minishlink\WebPush\VAPID;

class WebPushTestController extends Controller
{
    public function createKey(Request $request) {    
        $keyset = VAPID::createVapidKeys();
        dump($keyset);        
    }    
}
```

### 4. 구독 (http://localhost:8083/ 으로 접속하면 아래 코드가 실행되도록 하였다.)

```javascript
    <script src="/js/plugin/jquery.min.js"></script>
    <script>
        $(document).ready(function() {
            $.ajaxSetup({
                headers: {
                    'X-CSRF-TOKEN': $('input[name="_token"]').val()
                }
            });
        });

        $(function() {
            if(opener) {
                opener.location.reload();
                self.close();
            }

            Notification.requestPermission().then((status) => {
                console.log("Notification status : ", status);
                console.log('Url : ', location.protocol, location.host);

                if (status === "denied") {
                    console.log('denied');
                } 
                else if (navigator.serviceWorker) {
                    navigator.serviceWorker.register("/serviceworker.js?dummy=1") // register serviceworker
                        .then(function (registration) {
                            const pushkey = 'BCAY9s63tDicA9tEyPcHPQuCd3zaRB_bT..............EitTHEkakwlMOGHRYpPHsHGjoVPDV0_YoM' // $keyset 에서 생성된 public key 입력
                            const subscribeOptions = {
                                userVisibleOnly: true,                                
                                // https://developers.google.com/web/fundamentals/push-notifications/subscribing-a-user
                                applicationServerKey: pushkey,
                            };
                            $('#pushkey').text(pushkey);

                            return registration.pushManager.subscribe(subscribeOptions);
                        })
                        .then(function (pushSubscription) {       
                            // 서버로 전송                                                 
                            console.log('push', pushSubscription);
                            $.ajax({
                                // http://localhost:8083/push
                                url: location.protocol + '//' + location.host + '/push',
                                type: 'post',
                                data: JSON.stringify(pushSubscription),
                                contentType: 'application/json',
                                success: (res) => {
                                    console.log(res);
                                }
                            });

                        });
                }
            });

        });
    </script>
```
### 5. serviceworker.js

```php
self.addEventListener('push', (e) => {
    //const data = e.data.json()
    console.log('Push Received...', e);
    let dt = JSON.parse(e.data && e.data.text());
    e.waitUntil(
        self.registration.showNotification('title4', {
            body: dt.message || 'hello',
        })
    );
})


self.addEventListener("notificationclick", function (event) {
    event.notification.close();
    const urlToOpen = "https://naver.com";
    event.waitUntil(
        self.clients.matchAll({
            type: "window",
            includeUncontrolled: true,
        })
        .then(function (clientList) {
            if (clientList.length > 0) {
                // 이미 열려있는 탭이 있는 경우
                return clientList[0]
                    .focus()
                    .then((client) => client.navigate(urlToOpen));
            }
            return self.clients.openWindow(urlToOpen);
        })
    );
});

self.addEventListener('pushsubscriptionchange', function(event) {
    console.log('Subscription expired');
});

self.addEventListener('pushsubscriptionchange', function(event) {
    console.log('Subscription changed');
});
```

### 6. 구독 정보 저장

```php
namespace App\Http\Controllers;

use Minishlink\WebPush\WebPush;
use Minishlink\WebPush\Subscription;
use Minishlink\WebPush\VAPID;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Log;

class WebPushTestController extends Controller
{
    public function push(Request $request) {
        try {
            $param = $request->all();
            log::debug('endpoint success');
            log::debug(print_r($param, true));
            
            // 구독정보 상세
            /*
            Array
            (
                [endpoint] => https://fcm.googleapis.com/fcm/send/e0X81cT7Zko:APA91bEfNpjHTnOkz8D-IIdZoH1V-oGL7HthLUq-QdMovTsnRP5Blc3YB7obsvxW9cRXFCQOcJaXIBM4pNti4JxpfqYerhHTXQ2_csuHCXDsTpi9cXFk9lVNYa4aDMzsB2b60PXIGrhf
                [expirationTime] =>
                [keys] => Array
                    (
                        [p256dh] => BBXthEv1-JhhT3a7ucPtVpqAB5m2x077RPNZZV6_ixU2pnvyfoL2VrzqIN3o4njf9MCudWpGxq4Fp2VKvQmYjHo
                        [auth] => 0D3bIYtQSgQ5Auu6H8MWbQ
                    )

            )            
            */

            DB::table('tmp_push_queue')->insert([
                'endpoint' => $param['endpoint'],
                'auth_token' => $param['keys']['auth'],
                'p256dh' => $param['keys']['p256dh'],
                'msg' => 'Hello World',
            ]);

            return response()->json([
                'code' => true,
                'msg' => 'success',
                'data' => [],                
            ]);
        }
        catch(\Exception $e) {
            $this->writeErrorLog($e);
            return response()->json([
                'code' => false,
                'msg' => 'error',
                'data' => [],
            ]);
        }
    }  

    public function createKey(Request $request) {    
        $keyset = VAPID::createVapidKeys();
        dump($keyset);        
    }      
}
```

### 7. 웹 푸시 발송 컨트롤러 

```php
namespace App\Http\Controllers;

use Minishlink\WebPush\WebPush;
use Minishlink\WebPush\Subscription;
use Minishlink\WebPush\VAPID;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Log;

class WebPushTestController extends Controller
{
	public function sendPush(Request $request) {

        $queue = DB::table('tmp_push_queue')
            ->orderByDesc('id')
            ->first();

        dump($queue);

        // $keyset 에서 생성된 public key 및 private key 입력
        $publicKey = 'BCAY9s63tDicA9tEyPcHPQuCd3zaRB_bT..............EitTHEkakwlMOGHRYpPHsHGjoVPDV0_YoM';
        $privateKey = 'pBmVMr......xVET-ElS6Uw';

        $subscription = Subscription::create([
            'endpoint' => $queue->endpoint,
            'contentEncoding' => 'aesgcm',
            'authToken' => $queue->auth_token,
            'keys' => [
                'auth' => $queue->auth_token,
                'p256dh' => $queue->p256dh,
            ],            
        ]);

        $vapid = [
            'VAPID' => [
                'subject' => 'http://localhost:8083',
                'publicKey' => $publicKey,
                'privateKey' => $privateKey
            ]
        ];

        $webPush = new WebPush($vapid);
        $report = $webPush->sendOneNotification(
            $subscription,            
            json_encode([
                'message' => 'hello',
                'foo' => 'bar',
            ]),
        );

        $endpoint = $report->getRequest()->getUri()->__toString();

        if ($report->isSuccess()) {
            echo "[v] Message sent successfully for subscription<br/>";
            echo "{$endpoint}";
        } else {
            echo "[x] Message failed to sent for subscription<br/>";
            echo "{$endpoint}<br/>";
            echo "{$report->getReason()}";
        }
    }

    public function push(Request $request) {
        try {
            $param = $request->all();
            log::debug('endpoint success');
            log::debug(print_r($param, true));

            DB::table('tmp_push_queue')->insert([
                'endpoint' => $param['endpoint'],
                'auth_token' => $param['keys']['auth'],
                'p256dh' => $param['keys']['p256dh'],
                'msg' => 'Hello World',
            ]);

            return response()->json([
                'code' => true,
                'msg' => 'success',
                'data' => [],                
            ]);
        }
        catch(\Exception $e) {
            $this->writeErrorLog($e);
            return response()->json([
                'code' => false,
                'msg' => 'error',
                'data' => [],
            ]);
        }
    }  

    public function createKey(Request $request) {    
        $keyset = VAPID::createVapidKeys();
        dump($keyset);        
    }      
}
```

이제 브라우저로 http://localhost:8083/sendPush 로 접속하면 알림이 오는것을 확인할 수 있다. 만약 푸쉬가 오지 않는다면 OS 알림 또는 브라우저 알림 부분을 체크해봐야한다.