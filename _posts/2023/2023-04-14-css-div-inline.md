---
title: "Div 줄내림 방지(inline)"
categories:
- CSS
tags:
- style
- display
- inline
- inline-block
---

div 태그는 block element 이므로 별도의 CSS 없이는 줄 내림이 기본이다.

```php
<div>
    <div><h6>title</h6></div>
    <div>
        <button>button1</button>
        <button>button2</button>
    </div>
</div>
```

title 과 button 들을 일렬로 배치하려면 inline-block 사용하고, div 태그의 줄내림을 없애주면 가능하다. (div 태그 또한 캐리지 리턴(줄내림)없이 이어 작성했다는 점을 주목하자.)

```php
<div>
    <div style="display: inline-block; width: 50%;"><h6>title</h6></div><div style="display: inline-block; width: 50%;">
        <button>button1</button>
        <button>button2</button>
    </div>
</div>
```

위의 코드에서 div 를 첫번째 예제처름 줄내림을 하게되면 다시 원래대로 인라인이 풀리게된다. 

```php
<div>
    <div style="display: inline-block; width: 50%;"><h6>title</h6></div>
    <div style="display: inline-block; width: 50%;">
        <button>button1</button>
        <button>button2</button>
    </div>
</div>
```

이럴때 상위 element 에 다음과 같은 스타일을 주면 인라인 상태를 유지시킬수 있다.

```php
<div style="white-space:nowrap;">
    <div style="display: inline-block; width: 50%;"><h6>title</h6></div>
    <div style="display: inline-block; width: 50%;">
        <button>button1</button>
        <button>button2</button>
    </div>
</div>
```