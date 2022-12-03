---
title: "Javascript Functional Method in Array"
date: 2019-05-07T22:42:05+09:00
archives: "2019"
tags: ["javascript", "edwith", "webprogramming"]
author: Joohan Lee
---

## 배열의 함수형 메소드

### 1. for vs. forEach

- 기본 data set

```javascript
var data = [
    		{title: "Hello1", content: "random content1", price: 12000},
            {title: "Hello2", content: "random content2", price: 5500},
            {title: "Hello3", content: "random content3", price: 1200}
           ];
```

- for 사용

```javascript
for (var i = 0; i < data.length; i++) {
    console.log(data[i].title, data[i].price);
}
```

- forEach 사용 (함수형 Method 이용)

```javascript
data.forEach(function(v) {
    console.log(v.title, v.price);
});
```

<=>

```javascript
function printElement(v) {
    console.log(v.title, v.price);
}

data.forEach(printElement);
```

### 2. map

- 함수에 정의한 방법대로 모든 원소를 가공해서 새로운 배열을 반환
- ex) 10%가격을 인상시키고 싶다

```javascript
var newData = data.map(function(v) {
    return {title: v.title, content: v.content, price: v.price * 1.1};
})

console.table(data); // 원본 데이터
console.table(newData) // price가 10% 인상된 데이터
```

### 3. ilter

- 조건에 맞는 Element만 거르기

```javascript
var newData = data.filter(function(v) {
    return v.price > 5000;
});
```

- ex) Method chaining - price가 5000 이상인 data의 price에 000앞에 ","를 붙이고 맨 끝에"원" 붙이기

```javascript
var newData = data.filter(function(v) {
    return v.price > 5000;
}).map(function(v) {
  v.price = (''+v.price).replace(/^(\d+)(\d{3})$/, "$1,$2원");
  return v;
});
```

### 4. Immutable

- 데이터 값을 변경할 때 원본 데이터를 유지하기
- 어떤 이유로 원본데이터가 변경되면, 추적도 어렵고, 어디서 어떤 이유로 변경된 것인지 좀 알기 어렵고 다시 이전 상태의 값을 되돌려야 할 경우가 생김
- 이러한 경우 기존 데이터 값을 변경하는 것이 아니라 새롭게 객체를 생성해서 사용

```javascript
var filteredData = data.filter(function(v) {
    return v.price > 5000;
}).map(function(v) {
  var obj = {};
  obj.title = v.title;
  obj.content = v.content;
  obj.price = (''+v.price).replace(/^(\d+)(\d{3})$/, "$1,$2원");
  return obj;
});
```

### 5. reduce

- 배열의 모든 요소에 대해 지정된 Callback 함수를 호출하며 반환 값을 누적하여 반환하는 함수
- parameter로 Call back 함수와 누적을 시작하기 위한 초기 값을 넣고 며 초기 값은 선택사항

```javascript
var totalPrice = data.reduce(function(prevValue, product) { return prevValue + product.price; }, 0);

console.log(totalPrice);
```



원본 강의: https://www.edwith.org/boostcourse-web