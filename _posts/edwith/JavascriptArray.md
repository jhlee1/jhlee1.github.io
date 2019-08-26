---
title: "JavascriptArray"
date: 2019-02-27T21:58:12+09:00
archives: "2019"
tags: ["javascript", "edwith", "webProgramming"]
author: Joohan Lee
---

## Javascript Array

### 1. 배열 선언

```javascript
var arr = [];
var arr = [1, 2, 3, "hello", null, true, [[{1:0}]]];

console.log(arr.length); // 7
```

- new Array()로 선언할 수 도 있지만 대부분 []을 사용



### 2. 원소 추가

- index를 이용한 추가

```
var arr = [4];
arr[1000] = 3;
console.log(a.length); // 1001
console.log(arr[500]); // undefined
```

- push를 이용한 추가

```
var arr = [4];

arr.push(5);
console.log(arr.length); // 2
```



### 3. 기타 유용한 Method

```
[1,2,3,4].indexOf(3) // 2
[1,2,3,4].indexOf(7) // -1
[1,2,3,4].join() // "1,2,3,4"
[1,2,3,4].concat(2,3); // [1,2,3,4,2,3]

var origin = [1,2,3,4]

var result = [...origin,2,3];
console.log(result); // [1,2,3,4,2,3]
```

### 4. ForEach

- 아래 예제는 모두 같은 기능을 수행

```
result.forEach(function(v) {
    console.log(v);
})
```

```javascript
var func = function(v,i,o) {
    console.log(v);
};

result.forEach(func);
```

### 5. Map

```
var newArr = ["apple", "tomato"].map(function(value, index) {
    return index + "번째 과일은 " + value + "입니다";
});

console.log(newArr); // [ "0번째 과일은 apple입니다", "1번째 과일은 tomato입니다" ]
```



원본 강의: https://www.edwith.org/boostcourse-web