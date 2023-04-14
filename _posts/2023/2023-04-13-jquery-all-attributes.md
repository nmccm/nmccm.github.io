---
title: "Jquery 사용하여 특정 요소(element)의 모든 attributes 확인"
categories:
- JavaScript
- Jquery
tags:
- JavaScript
- Jquery
---

Jquery 사용하여 특정 요소(element)의 모든 attributes 를 가져오는 코드

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
