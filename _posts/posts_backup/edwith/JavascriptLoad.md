---
title: "Javascript DOM ContentLoaded Event"
date: 2019-04-16T13:25:24+09:00
archives: "2019"
tags: ["edwith", "javascript", "webProgramming"]
author: Joohan Lee
---

##  DOM ContentLoaded Event 와 Window.load의 차이


### 1. DOM Content Loaded와 load의 차이

- 크롬 개발자도구의 Network panel을 열어서 왼쪽 아래에 **DOMContentLoaded, load**를 확인
- DOM Tree 분석이 끝나면 DOMContentLoaded 이벤트가 발생하며, 그 외 모든 자원이 다 받아져서 브라우저에 렌더링(화면 표시)까지 다 끝난 시점에는 Load가 발생
- 보통 DOM tree가 다 만들어지면 DOM APIs를 통해서 DOM에 접근할 수 있기 때문에, 실제로 실무에서는 대부분의 자바스크립트코드는 DOMContentLoaded 이후에 동작하도록 구현하기 때문에 load는 거의 쓰이지 않음

```javascript
//DOM Content Loaded Event 추가
document.addEventListener("DOMContentLoaded", function() {
  console.log("DOM is loaded.");
});

//Window load Event 추가
document.addEventListener("load", function() {
  console.log("the page is loaded.");
});
```

원본 강의: https://www.edwith.org/boostcourse-web
