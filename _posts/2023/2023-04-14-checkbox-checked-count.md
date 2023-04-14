---
title: "HTML Checkbox library"
categories:
- JavaScript
- Jquery
tags:
- JavaScript
- Jquery
---

Html 그리드(Grid)에서 첫번째 행에 체크박스를 두어 "선택삭제", "선택수정" 등등의 액션을 사용자가 취할 수 있도록 UI 구성 하는것이 일반적이다.
이 체크박스가 선택이 되었는지 여부를 체크하는 모듈을 만들고 재사용함에 목적을 둔다. (코딩량은 아주 조금(?) 감소)  

```html
<table style="border: 1px solid red;">
    <tr>
        <td><input type="checkbox" name="bc" data-order-id="1" data-brand-id="3"></td>
        <td>11111</td>
    </tr>
    <tr>
        <td><input type="checkbox" name="bc" data-order-id="2" data-brand-id="4"></td>
        <td>11111</td>
    </tr>
</table>
<div>
    <button id="submit">submit</button>
</div>
```

```javascript
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

window.erp = window.erp || {};

window.erp.checkbox = {
    selector: {},
    datas: {},
    getCheckedLength: () => {
        let cnt = 0;
        
        erp.checkbox.selector.each(function() {
            if($(this).is(':checked')) {
                ++cnt;
            }
        });
        
        if(cnt > 0) {
            erp.checkbox.selector.each(function() {
                if($(this).is(':checked')) {
                    const attr = $(this).attr();
                    for(let key in attr) {
                        if(key.toString().indexOf('data-') > -1) {
                            let prop = key.toString().replace('data-', '');
                            erp.checkbox.datas[prop] = attr[key];
                        }
                    }
                }
            });
        }
        return cnt;
    },
    getDataProperties: () => {
        return erp.checkbox.datas;
    }
};
  
$(function() {
    $("#submit").on('click', function(e) {
        
        e.preventDefault();
        const cb = erp.checkbox;
        cb.selector = $("input[name=bc]");

        if(cb.getCheckedLength() <= 0) {
            alert('항목을 선택해주세요.');
            return false;
        }

        if(cb.getCheckedLength() > 1) {
            alert('하나의 항목만 선택해주세요');
            return false;
        }

        let orderId = cb.getDataProperties()['order-id'];
        let brandId = cb.getDataProperties()['brand-id'];
        
        console.log(orderId, brandId);
    });
});
</script>
```

