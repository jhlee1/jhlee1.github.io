---
title: "JavascriptEventDelegation"
date: 2019-04-16T13:28:33+09:00
archives: "2019"
tags: ["javascript", "edwith", "webProgramming"]
author: Joohan Lee
---

## Javascript Event Delegation

### 1. 각각의 li 마다 event listener를 등록

- main.html

```html
<html>
	<head>
        <link rel="stylesheet" href="./css/main.css">
    </head>    
    <body>
        <ul>
  			<li>
    			<img src="https://images-na.,,,,,/513hgbYgL._AC_SY400_.jpg" class="product-image" >    </li>
			<li>
    			<img src="https://images-n,,,,,/41HoczB2L._AC_SY400_.jpg" class="product-image" >    </li>
			<li>
    			<img src="https://images-na.,,,,51AEisFiL._AC_SY400_.jpg" class="product-image" >  </li>
			<li>
    			<img src="https://images-na,,,,/51JVpV3ZL._AC_SY400_.jpg" class="product-image" >
 			</li>
		</ul>
    </body>
    <section class="log"></section>
    <script src="js/main.js"></script>
</html>
```

- main.js

```javascript
var log = document.querySelector(".log");
var lists = document.querySelector("ul > li");

for (var i = 0; len = lists.length; i < lists.length; i++) {
    lists[i].addEventListener("click", function(evt) {
        console.log(evt.currentTarger);
        log.innerHTML = "IMG URL is" + evt.currentTarget.firstElementChild.src;
    });
}
```

- 브라우저는 4개의 이벤트 리스너를 기억하고 있다. <li>가 늘어날수록 브라우저는 기억해야 할 이벤트 리스너도 그만큼 많아진다. -> 비효율적
- 추가적으로 <li>가 한 개 더 동적으로 추가된다면 또  addEventListener를 해줘야 한다.



### 2. ul 태그에만 이벤트를 새롭게 등록 

- currentTarget 대신 target을 사용

```javascript
var log = document.querySelector(".log");
var ul = document.querySelector("ul");

ul.addEventListener("click" function(evt) {
    console.log(evt.target.tagName, evt.currentTarget.tagName); //target = IMG , currentTarget = UL
})
```

### 3. Event Bubbling

- <img>도 클릭 이벤트 등록하면 li 나 img 태그는 ul 태그에 속하기 때문에 클릭 시 둘 다 Event가 발생
- 이벤트 버블링 - 클릭한 지점이 하위엘리먼트라고 하여도, 그것을 감싸고 있는 상위 엘리먼트까지 올라가면서 이벤트리스너가 있는지 찾는 과정
- 만약 img, li, ul에 각각 이벤트를 등록했었다면, 3개의 이벤트 리스너가 실행됨 
- img > li > ul 순으로 이벤트가 발생
- 기본적으로는 Bubbling 순서로 이벤트가 발생하지만 반대 순서로 발생하도록 할 수 있다. 이러한 경우를 Event Capturing이라고 부른다.
- Capturing 단계에서 이벤트 발생을 시키고 싶다면 addEventListener 메서드의 3번째 인자에 값을 true로 주면 됨

```javascript
var log = document.querySelector(".log");
var ul = document.querySelector("ul");
var img = document.querySelector("img");

img.addEventListener("click", function() {
    console.log("img tag clicked!!");
})

ul.addEventListener("click" function(evt) {
    console.log(evt.target.tagName, evt.currentTarget.tagName); //target = IMG , currentTarget = UL
})

// console log
// img tag clicked!!
// IMG UL
```

### 4. <img>와 <li>를 클릭했을 때 img url 을 출력하기

- parent element에다 Click Event를 위임(delegation)하기 때문에 Event Delegation이라고 부름

```javascript
var log = document.querySelector(".log");
var ul = document.querySelector("ul");

ul.addEventListner("click", function(evt) {
    var target = evt.target;
    
    if(target.tagName === "IMG") {
        log.innerHTML = "IMG URL is: " + target.src;
    } else if (target.tagName === "LI") {
        log.innerHTML = "IMG URL is: " + target.firstElementChild.src;
    }    
});

```

원본 강의: https://www.edwith.org/boostcourse-web