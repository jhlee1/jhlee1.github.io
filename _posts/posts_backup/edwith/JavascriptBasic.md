---
title: "JavascriptBasic"
date: 2019-01-31T22:35:29+09:00
archives: "2019"
tags: ["javascript", "edwith", "webProgramming"]
author: Joohan Lee
---

1. Javascript Hoisitng
- 함수는 실행되기 전 함수 안의 필요한 변수값들을 미리 다 모아서 선언

ex1)
```Javascript
function outer() {
    var result = inner;

    console.log(result) //result: undefined error

    var inner = function () {
        return "inner value";
    }
}
```
inner의 내부값이 assign되기 전으로 처리되어 에러 발생.

실제 처리된 결과:
```Javascript
function outer() {
    var result;
    var inner;

    console.log(result);

    inner = function () {
        return "inner value";
    }
}
```

ex2)
```Javascript
function outer() {
    var result = inner();

    console.log(result); // result: inner value

    function inner() {
        return "inner value";
    }
}
```
Hoisting을 통해 함수가 위에서 선언되어 정상처리됨.

실제 처리된 결과:
```Javascript
function outer() {
    function inner() {
        return "inner value";
    }

    var result = inner();

    console.log(result);
}
```


2. Arguments
- 함수내에  arguments로 지역변수가 자동으로 생성
- arguments의 타입은 Object 입니다.(Array가 아님)
- function의 signature에 선언한 parameters보다 더 많은 값을 보낼 수 있다.
- 받아온 parameters를 arguments로 array의 형태로 하나씩 접근할 수 있지만 array의 메서드를 사용할 수가 없다. (object이기 때문에)

ex)
```javascript
function list() {
    console.log(arguments); // {'0':1, '1':2, '2':3, '3':4}
    for(int i = 0; i < arguments.length; i++) {
        console.log(arguments[i]);
    }
}

list(1,2,3,4);
```

3. arrow function
- Java의 lamda와 비슷한 느낌이다.
```Javascript
function getName(name) {
    return "Mr." + name;
}
```

위의 코드는 아래와 같은 역할

```
var getName = (name) => "Mr." + name;
```

4. 함수의 Call Stack
- 브라우저는 Call Stack의 개수를 제한 해 놓는다. 제한을 넘어서 Call Stack을 쌓을 경우 Maximum call stack size exceeded 에러가 발생
- [참고 링크](https://medium.com/@gaurav.pandvia/understanding-javascript-function-executions-tasks-event-loop-call-stack-more-part-1-5683dea1f5ec)

5. Window 객체
- 브라우저를 개발하다보면 window라는 객체가 있다.
- 많은 method를 가지고 있다.
ex)

```Javascript
window.setTimeout()
setTimeout // window는 default 개념임으로 생략 가능
```

- setTimeout 활용
  - 인자로 함수를 받고 있으며, 나중에 실행되는 함수를 Callback 함수라고 함

- ex)
```Javascript
function run() {
  console.log("run start")
  setTimeout (function() {
    var msg = "hello world!";
    console.log(msg);
    console.log("run ...ing");
  }, 1000)

  console.log("run end!");
}  
```
- Result
```
run start
run end!
hello world!
run ...ing
```

- setTimeout 안의 Callback 함수는 run함수의 실행이 끝나고 나서 , (정확히는 stack에 쌓여있는 함수의 실행이 끝나고 나서 실행됨) 실행됩니다
- 비동기 함수는 나중에 처리되도록 stack에 쌓였다가 처리됨

ex2)
```Javascript
function run() {
  setTimeout (function() {
    var msg = "hello world!";
    console.log(msg);
  }, 1000)
  console.log("run function end");
}

console.log("run start")
run()
console.log("run end!");
```
- Result
```
run start
run function end
run end!
hello world!
```

6. DOM(Document Object Model)

- Browser는 HTML코드를 DOM Tree로 저장함
- HTML코드가 화면상에 한번 출력되고 끝나는게 아니라 계속 업데이트 되므로 DOM Tree가 업데이트 되는 부분을 찾기 쉬움
- 복잠한 DOM Tree를 Javascript로 탐색 알고리즘을 이용하기는 비효율적 -> Browser가 DOM API를 제공
  - getElementById()
    - ID 정보를 통해서 찾을 수 있음.
    ex)
    ```Javascript
    //개발자 도구를 이용해서 console에서 입력해보기
    > document.getElementById("example").style.display = "none";
    < "none"
    //해당 Element가 화면에서 사라짐
    ```
  - querySelector
    - CSS selector를 기반으로 Element를 찾아줌

    ex)
    ```Javascript
    document.querySelector("div");
    document.querySelector("body #example1");
    document.querySelector(".example2 > .example3");
    ```

  - querySelectorAll 해당하는 모든 Element를 반환

7. Event
- 브라우저의 화면에는 많은 Event(마우스 이동, 클릭, 스크롤, 키보드 입력 등등)들이 발생
- Event가 발생했을 때 특정 행동을 하도록하기 위해 해당 Element를 Javascipt를 통해 등록하여 처리할 수 있다.
- addEventListener
  - Event가 발생될 때 실행되는 함수로, Event Handler 또는 Event Listener라고 불림.
  - Parameter를 통해 Event Object를 받을 수 있음.
ex)
```Javascript
var el = document.getElementById("outside");
el.addEventListener("click", function() {
  //do something
}, false);

var el = querySelector(".outside");
el.addEventListener("click", function(e) {
  console.log("clicked!!", e);
  console.log("clicked!!", e.target);
  console.log("clicked!!", e.target.nodeName);
}, false);
```

8. Ajax(XMLHTTPRequest 통신)

- 새로고침 없이 비동기로 처리되는 요청
- JSON
  - 서버와 클라이언트가 정보를 주고 받기 위한 형식

ex)
```Javascipt
function ajax(data) {
  var oReq = new XMLHTTPRequest();
  oReq.addEventListener("load", function() {
      console.log(this.responseText);
  });

  oReq.open("GET", "http://www.example.org/getData?data=data")
  oReq.send();
}
```
XMLHTTPRequest 객체 생성 -> open으로 Request 보낼 준비 -> send로 Request를 보냄 -> Response가 오면 "load" Event 발생 -> Callback 함수가 실행됨
- [참고 자료] CORS 찾아보기

9. Debugging
- Framework 없이 실행될 수 있는 것을 Vanilla JS라고 부름

원본 강의: https://www.edwith.org/boostcourse-web
