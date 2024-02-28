---
title: "Jquery 사용하여 특정 요소(element)의 모든 attributes 확인"
categories:
- JavaScript
tags:
- Jquery
---

### 1. 환경

```bash
```

### 2. 배경

사이트 개발을 진행하면서 자바스크립트 모듈화를 진행하다보면 특성 셀렉터 또는 요소의 모든 속성(attributes) 확인이 필요한 상황이 발생한다. 그에 대한 대응 코드이다.

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
