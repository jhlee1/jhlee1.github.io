---
title: "Object literal and this"
date: 2019-05-07T23:35:35+09:00
archives: "2019"
tags: ["edwith", "javascript", "webProgramming"]
author: Joohan Lee
---

## Object Literal과 this

### 1. 자바스크립트 객체의 활용

- 객체 리터럴 예제

```javascript
var healthObj = {
  name : "달리기",
  lastTime : "PM10:12",
  showHealth : function() {
    console.log(this.name + "님, 오늘은 " + this.lastTime + "에 운동을 하셨네요"); // this는 이 함수가 불리는 Context를 가르킴. healthObj를 this로 접근
  }
}

healthObj.showHealth();
```

### 2. this

- 객체 안에서의 this는 그 객체 자신
- ES6에서는 객체에서 메서드를 사용할 때 'function' 키워드를 생략할 수 있다.

```javascript
const obj = {
   getName() {
     return this.name;
     },
  setName(name) {
      this.name = name;
    }
}
obj.setName("crong");
const result = obj.getName();
```

- JavaScript에는 전역스크립트나 함수가 실행될 때 실행문맥(Execution context)이 생성
- (브라우저내에 stack에 올라가서 실행됨)
- 모든 context에는 참조하고 있는 객체(thisBinding이라 함)가 존재하는데, 현재 context가 참조하고 있는 객체를 알기 위해서는 this를 사용
- 즉, 함수가 실행될때 함수에서 가리키는 this 키워드를 출력해보면 context가 참조하고 있는 객체를 알 수 있음

```javascript
function get() {
    return this;
}

get(); //window. 함수가 실행될때의 컨텍스트는 window를 참조한다.
new get(); //object. new키워드를 쓰면 새로운 object context가 생성된다.
```

- method안의 this가 항상 해당 객체를 가르키는 건 아니다. 임의적으로 context가 변경될 수 있다.

```javascript
var other = {
	todos: "I don't do anything";
}

var todo = {
	todos: ["Study JS"],
	addTodo: function(newTodo) {
		this.todos.push(newTodo);
	},
	showTodo: function() {
		return this.todos;
	}
}

todo.showTodo(); // ["Study JS"]
todo.showTodo.call(others); // "I don't do anything"
```

### 3. bind 이용하기

- this가 달라지는 경우
- 아래의 경우 this는 setTimeout에 의해 context가 변경되어  Window 객체를 가르킴 (debugger를 추가하여 검사)

```javascript
var healthObj = {
  name : "달리기",
  lastTime : "PM10:12",
  showHealth : function() {
    setTimeout(function() {
        console.log(this.name + "님, 오늘은 " + this.lastTime + "에 운동을 하셨네요");      
    }, 1000)
  }
}
healthObj.showHealth(); // 님, 오늘은 undefined에 운동을 하셨네요
```

- bind는 **새로운 함수를 반환**하는 함수
- bind함수의 첫번째 인자로 this를 주어, 그 시점의 this를 기억하고 있는 새로운 (바인드된)함수가 반환되는 것
- 아래의 경우 bind를 사용하여 원하는 결과를 출력

```javascript
var healthObj = {
  name : "달리기",
  lastTime : "PM10:12",
  showHealth : function() {
    setTimeout(function() {
        console.log(this.name + "님, 오늘은 " + this.lastTime + "에 운동을 하셨네요");      
    }.bind(this), 1000)
  }
}
healthObj.showHealth(); // 달리기님, 오늘은 PM10:12에 운동을 하셨네요
```

- ES6의 Arrow 함수를 사용할 경우 bind를 사용하지 않아도 됨(context가 유지됨)
- 이 경우 window를 쓰려면 this 대신 직접 window를 쓰거나 bind를 해줘야함

```javascri
var healthObj = {
  name : "달리기",
  lastTime : "PM10:12",
  showHealth : function() {
    setTimeout(() => {
        console.log(this.name + "님, 오늘은 " + this.lastTime + "에 운동을 하셨네요");      
    }, 1000)
  }
}
healthObj.showHealth(); // 달리기님, 오늘은 PM10:12에 운동을 하셨네요
```



원본 강의: https://www.edwith.org/boostcourse-web
