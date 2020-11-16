---
title: "JavascriptAsync"
date: 2019-04-12T17:08:52+09:00
archives: "2019"
tags: ["javascript", "edwith", "webProgramming"]
author: Joohan Lee
---

## 1. AJAX와 비동기

```javascript
function ajax() {
    var oReq = new XMLHttpRequest();
    
    oReq.addEventListener("load", function() { //밑에 oReq.open과 oReq.send보다 더 늦게 실행됨.
        console.log(this.responseText);
    });
    
    oReq.open("GET", "http://www.example.org/example/txt");
    oReq.send();
}
```

- 비동기로 실행되는 EventListener속에 있는 anonymous 함수는 브라우저가 가지고 있는 Web Apis와 Callback Queue(=task queue)를 거쳐서 다시 Call Stack으로 들어가기 때문에 가장 나중에 실행됨.(Call stack -> Web Apis -> Callback Queue -> Call stack이 비어있는지 확인 후 Call Stack으로 이동 후 -> execute의 단계를 거침)

- 동기로 실행되는 함수들의 경우 바로 Call Stack에서 실행됨
- 참고 링크: <https://www.youtube.com/watch?v=8aGhZQkoFbQ>

### 3. Ajax 응답 처리

- 위에서 받은 데이터가 JSON 형식의 text로 반환된다면  아래와 같이 parsing처리하면 됨.

```javascript
var jsonObject = JSON.parse("{"name": "tester1", "age": 17}")
```

### 4. Cross Domain 문제

- XHR통신은 보안상 다른 도메인 간에 통신 불가능하기
- exampleA.com <-> exampleB.com XHR통신, Ajax 통신 불가능
- 이를 회피하기 위해 JSONP(비표준) 와 CORS(표준) 방식을 사용하고 있다.
- JSONP은 비표준이지만 아직도 많은 곳에서 CORS로 넘어가지 않고 사용하고 있기 때문에 사실상 표준으로 인식되고 있다. 
- CORS는 BackEnd코드에서 header를 설정해줘야하는 번거로움이 존재한다.



원본 강의: https://www.edwith.org/boostcourse-web