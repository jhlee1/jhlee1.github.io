---
title: "JavascriptAnimation"
date: 2019-04-13T01:25:24+09:00
archives: "2019"
tags: []
author: John SMITH
---

## Animation과 SetTimeout의 이해

### 1. Animation

- 반복적인 움직임의 처리
- Web UI의 animation은 Javascript로 다양하게 처리할 수 있지만, 규칙적이고 단순한 방식으로 동작되는 것은 CSS3의 transition과 transform 속성을 이용하여 구현 가능
- CSS를 사용하는 것이 Javascript 보다 더 빠른 성능을 보여줌
- **특히,** 모바일 웹에선 CSS를 사용하는 것이 훨씬 더 빠름
- 간단하고 규칙적이다 -> CSS로 구현
- 세밀하고 조작이 필요하다 -> JS로 구현
- 성능만 봐서는 대체로 CSS가 빠르나 성능 비교를 통해서 가장 빠른 방법을 찾는 과정이 필요

### 2. FPS (Frame Per Seconds)

- 1초에 화면에 표현할 수 있는 정지화면(frame)의 수
- 매끄러운 애니메이션은 보통 60fps
- 16.666초 간격으로 frame 생성 (1000ms / 60 fps = 16.6666ms)

### 3. JavaScript Animation

- Javascript로 구현하려면, 규칙적인 처리를 하도록 구현하면 됨
- 아래의 방법을 사용
  - setInterval
  - setTimeout
  - requestAnimationframe
  - Animations API

#### 3.1 setInterval()

- 주어진 시간에 따라 함수 실행

```javascript
const interval = window.setInterval(()=> {
  console.log('현재시각은', new Date());
},1000/60);

window.clearInterval(interval);
```

- 비동기 작업이기 때문에 중간에 Mouse Click Event등이 발생하면 제 때 일어나야할 이벤트 콜백이 밀리게되어서 지연되고, 없어질 수 있음
- 따라서, setInterval을 사용한다고 정해진 시간에 함수가 실행된다고 보장할 수 없다.
- 위의 이유 때문에 setInterval을 사용하는 애니메이션을 잘 구현하지 않는다.

#### 3.2 setTimeout()

- 주어진 시간 후에 실행

```javascript
setTimeout(() => {
    console.log('현재시각은', new Date());
},500); //500ms 마다 현재 시각을 알려줌
```

- Animation 구현을 위한 Recursive Call

```javascript
let count = 0;
function animate() {   
  setTimeout(() => {
    if(count >= 20) return;
    console.log('현재시각은', new Date());
    count++;
    animate();
  },500);
}
animate();
```

- setTimeout도 중간에 Event가 발생하면 정확히 500ms마다 실행이 안될 수 있다. 
- 하지만, setTimeout은 하나의 callback이 끝나고 다음 callback이 실행되기 때문에 setInterval처럼 Queue가 가득차서 중간에 사라지는 일이 없다.
- 또한 setTimeout은 매 순간 timeout을 조절할 수 있는 코드구현으로 컨트롤 가능한 부분이 있다는 점이 setInterval과 큰 차이

#### 3.3 requestAnimationFrame

- setTimeout을 이용한 Recursive call의 경우 animation을 위한 최적화된 기능이 아니다. animation주기를 16.6 미만으로 하는 경우 불필요한 frame 생성이 되는 등의 문제가 발생하기 때문에 requestAnimationFrame을 사용해야한다.

```html
<html>
    <header>
    	<style>
            .outside {
                position: relative;
                background-color: blueviolet;
                width: 200px;
                font-size: 0.8em;
                color: #fff;
            }
        </style>
    </header>
    <body>
      <div class="outside">The fruit I like the most is: </div>
    </body>
</html>
```

```javascript
var count = 50;
var el = document.querySelector(".outside");
el.style.left = "0px";

function run() {
	if (count > 50) return;
	count += 1;
	el.style.left = parseInt(el.style.left) + count + "px";
	requestAnimationFrame(run);
}

requestAnimationFrame(run);
```

### 4. CSS를 이용하여 Animation 구현

- transition을 이용한 방법
  - 이 방법이 JS를 이용하는 것보다 더 빠르다고 알려져있다. 특히 모바일웹에선 transform을 이용한 element조작을 많이 구현함
  - <https://thoughtbot.com/blog/transitions-and-transforms>

```html
<html>
    <header>
    	<style>
            .outside {
                position: relative;
                background-color: blueviolet;
                width: 200px;
                font-size: 0.8em;
                color: #fff;
                left: 100px;
                top: 100px;
                transform: scale(2);
                transition: transform 2s; // transform에 대해서만 2초 동안 transition 반응
                // transition: all 2s; 모든 CSS 변경에 2초 동안 transition 반응
            }
        </style>
    </header>
    <body>
      <div class="outside">The fruit I like the most is: </div>
        <button>
            right!
        </button>
    </body>
    <script src="test.js"></script>
</html>
```

```javascript
var base = document.querySelector(".outside");
base.style.transform = "scale(4)";
base.style.left = "500px";
```

- transform에는 transition이 실행되지만 left 변경에는 실행되지 않음



- 추가 예제

```html
<html>
    <header>
    	<style>
        .outside {
          position: relative;
          background-color: blueviolet;
          width: 200px;
          font-size: 0.8em;
          color: #fff;
          left: 100px;
          top: 100px;
          transform: scale(1);
          transition: left 0.5s ease-in;
        }
      </style>
    </header>
    <body>
      <div class="outside" style="left:100px;">The fruit I like the most is: </div>
        <button>
            right!
        </button>
    </body>
    <script src="test.js"></script>
</html>

```

```javascript
var target = document.querySelector(".outside");
var btn = document.querySelector("button");

btn.addEventListener("click", function() {
    var pre = parseInt(target.style.left);
    target.style.left = pre + 100 + "px";
});
```

#### 4.1 더 빠른 CSS3 애니메이션

- GPU 가속을 이용하는 속성을 사용하면 애니메이션 처리가 빠름
  - transform: translateXX();
  - transform: scale();
  - transform: rotate();
  - opacity
  - <https://d2.naver.com/helloworld/2061385>

### 5. To do in the future

1. 특정 엘리먼트를 만들고, 그 엘리먼트에 transition 속성을 주고, hover했을 때 좌측 상단에서, 우측 하단으로 움직이는 animation을 만들어보세요.
2. vendor prefix가 무엇이고, 왜 필요한지 알아봅니다.



원본 강의: https://www.edwith.org/boostcourse-web