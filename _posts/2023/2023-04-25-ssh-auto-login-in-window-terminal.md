---
title: "윈도우 터미널(Window Terminal)에서 ID/PW 사용한 SSH 자동 로그인(보안 취약 주의)"
categories:
- Window
tags:
- Terminal
- Shell
---

가볍고, 휴대성 좋고, 막강한 기능을 자랑하는 Putty 를 이용하여 리눅스 쉘 접속을 자주 하는 편이다. 이 편리한 Putty 이용하여 자동 로그인을 할 순 없을까 생각하여 찾아본 결과 스텍오버플로우에서 좋은 글을 발견하여 여기에 기록한다.

putty 에 로그인 할 계정을 save 버튼을 이용하여 저장한다.
- ex) 8.8.8.8-live-web

윈도우 터미널에서 해당 호스트를 로드하여 실행시킨다.

```php
putty -load "8.8.8.8-live-web" -l my_id -pw my_password
```

