---
title: "Jquery 사용하여 특정 요소(element)의 모든 attributes 확인"
categories:
- JavaScript
tags:
- Jquery
---

언제부턴가 HTML 태그내에 사용자의 입력값과 함께 전송될 정보(테이블 Primary Key 등등)들을 구분하게 되었다. Form 을 그대로 전송하는 코드라면 hidden 에 넣어 전송하면 되지만 그렇지 않고, 비동기로 구현되어야 하는 부분 즉 Form 을 사용하기 힘든 부분에서는 data- 프로퍼티를 많이 사용하게 되었다. 아래 코드는 data- 포함한 모든 어트리뷰트(attribute)를 가져오는 코드이다.

### 1. 환경

```bash
```

### 3. 코드

```php
<script>
    (function(old) {
        $.fn.attr = function() {
            if(arguments.length === 0) {
                if(this.length === 0) {
                    return null;
                }
                let obj = {};
                $.each(this[0].attributes, function() {
                    if(this.specified) {
                        obj[this.name] = this.value;
                    }
                });
                return obj;
            }
            return old.apply(this, arguments);
        };
    })($.fn.attr);
    
    $(function() {
        const attrs = $('#id').attr();
        console.log(attrs);    
    });
</script>
```
