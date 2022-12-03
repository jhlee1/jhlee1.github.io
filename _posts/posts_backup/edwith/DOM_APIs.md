---
title: "DOM APIs"
date: 2019-04-12T10:53:30+09:00
archives: "2019"
tags: ["javascript", "edwith", "webProgramming"]
author: John SMITH
---

## DOM API 활용

### 1. 다양한 APIs
- [document. 관련API](https://www.w3schools.com/jsref/dom_obj_document.asp)
- [element. 관련 API](https://www.w3schools.com/jsref/dom_obj_all.asp)

### 2. DOM 엘리먼트 오브젝트

- tagName : Element의 tag name
- textContent :  Node의 text content를 설정하거나 얻어옴
- nodeType : Node type을 숫자로 나타냄
- childNodes: Element의 Childs를 Node List로 나타냄 (Text node, Comment node 포함)
  - ex)
```javascript
Javascipt 코드
var ul = document.querySelector("ul");
var ulChildNodes = ul.childNodes

for (var index = 0; index < ulChildNodes.length; index++) {
  // 개행이 포함되어 textNode 포함
  console.log(ulChildNodes[index].nodeName);
}

console 결과
"#text"
"LI"
"#text"
"LI"
"#text"
"LI"
"#text"
```
- firstChild: Element의 첫 번째 공백이나 Text Node를 Child로 인식하여 나타냄
  - ex)
```javascript
Javascipt 코드
var body = document.querySelector("body");
var bodyFirstNode = body.firstChild;

// 개행이 되었므로 textNode가 선택
console.log(bodyFirstNode.nodeName);

console 결과
"#text"
```
- firstElementChild: 공백이나 Text Node를 무시하고 첫번째 Element를 나타냄
  - ex)
```javascript
Javascript 코드
var body = document.querySelector("body");
var bodyFirstElementChild = body.firstElementChild;

console.log(bodyFirstElementChild.nodeName);

console 결과

"DIV"
```

- parentElement: Element의 Parent Element를 반환 
  - ex)
```javascript
Javascipt 코드
var div = document.querySelector("div");
var divParentElement = div.parentElement;

console.log(divParentElement.nodeName);

console 결과
"BODY"
```
- nextSibling: 동일한 Node Tree 레벨에서 다음 **Node**를 반환 
  - ex)
```javascript
Javascipt 코드
var list1 = document.getElementById("list1");
var nextSiblingNode = list1.nextSibling;

// 개행문자로 인해 textNode 선택
console.log(nextSiblingNode.nodeName);

console 결과
"#text"
```
- nextElementSibling: 동일한 Node Tree 레벨에서 다음 **Element**를 반환 
  - ex)
```javascipt
Javascript 코드
var list1 = document.getElementById("list1");
var nextSiblingElement = list1.nextElementSibling;

console.log(nextSiblingElement.textContent);

console 결과
"2. DB 연결 웹 애플리케이션"
```

### 예제 HTML 기본 코드

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width">
  <title>부트스트랩</title>
</head>
<body>
  <div>
    <ul>
      <li>1. 웹 프로그래밍 기초</li>
      <li>2. DB연결 웹 애플리케이션</li>
      <li>3. 웹 애플리케이션 개발 1/4</li>
    </ul>
  </div>
 </html>
```



### 3. DOM 조작 API

- 삭제, 추가, 이동, 교체를 위해 아래의 API들을 사용
- removeChild(): Element의 Child Node 제거
  - ex)

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width">
  <title>부트스트랩</title>
</head>
<body>
  <div>
    <ul>
      <li id="list1">1. 웹 프로그래밍 기초</li>
      <li id="list2">2. DB연결 웹 애플리케이션</li>
      <li id="list3">3. 웹 애플리케이션 개발 1/4</li>
    </ul>
  </div>
  <button id="removeChildButton">div 자식 노드 제거</button>
</body>
</html>
```

```javascript
var removeChildButton = document.getElementById("removeChildButton");

removeChildButton.addEventListener("click", function(){
  var ul = document.querySelector("ul");
  
  ul.removeChild(ul.firstElementChild);
});
```

- appendChild(): 마지막 Child Node로 추가
  - ex)

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width">
  <title>부트스트랩</title>
</head>
<body>
  <div>
    <ul>
      <li id="list1">1. 웹 프로그래밍 기초</li>
      <li id="list2">2. DB연결 웹 애플리케이션</li>
      <li id="list3">3. 웹 애플리케이션 개발 1/4</li>
    </ul>
  </div>
  <button id="appendChildButton">ul 자식 노드 추가</button>
</body>
</html>
```

```javascript
var appendChildButton = document.getElementById("appendChildButton");

appendChildButton.addEventListener("click", function(){
  var ul = document.querySelector("ul");
  var liNode = document.createElement("LI");
  var textNode = document.createTextNode("4. 웹 애플리케이션 개발 1/4");
  
  liNode.appendChild(textNode);
  ul.appendChild(liNode);
});
```

- insertBefore(): 기존 Child Node 앞에 새 Child Node를 추가
  - ex)

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width">
  <title>부트스트랩</title>
</head>
<body>
  <div>
    <ul>
      <li id="list1">1. 웹 프로그래밍 기초</li>
      <li id="list2">2. DB연결 웹 애플리케이션</li>
      <li id="list3">3. 웹 애플리케이션 개발 1/4</li>
    </ul>
  </div>
  <button id="insertBeforeButton">ul 자식 노드 추가</button>
</body>
</html>
```

```javascript
var insertBeforeButton = document.getElementById("insertBeforeButton");

insertBeforeButton.addEventListener("click", function(){
  var ul = document.querySelector("ul");
  var liNode = document.createElement("LI");
  var textNode = document.createTextNode("4. 웹 애플리케이션 개발 1/4");
  
  liNode.appendChild(textNode);
  ul.insertBefore(liNode, ul.firstElementChild);
});
```

- cloneNode(): Node 복제
  - ex)

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width">
  <title>부트스트랩</title>
</head>
<body>
  <div>
    <ul>
      <li id="list1">1. 웹 프로그래밍 기초</li>
      <li id="list2">2. DB연결 웹 애플리케이션</li>
      <li id="list3">3. 웹 애플리케이션 개발 1/4</li>
    </ul>
  </div>
  <button id="insertBeforeButton">ul 자식 노드 추가</button>
</body>
</html>
```

```javascript
var insertBeforeButton = document.getElementById("insertBeforeButton");

insertBeforeButton.addEventListener("click", function(){
  var ul = document.querySelector("ul");

   ul.appendChild(ul.firstElementChild.cloneNode(true));
});
```

- replaceChild(): Replace Node
  - ex)

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width">
  <title>부트스트랩</title>
</head>
<body>
  <div>
    <ul>
      <li id="list1">1. 웹 프로그래밍 기초</li>
      <li id="list2">2. DB연결 웹 애플리케이션</li>
      <li id="list3">3. 웹 애플리케이션 개발 1/4</li>
    </ul>
  </div>
  <button id="replaceTextNodeButton">textNode 변경</button>
</body>
</html>
```

```javascript
var replaceTextNodeButton = document.getElementById("replaceTextNodeButton");

replaceTextNodeButton.addEventListener("click", function(){
  var ul = document.querySelector("ul");
  var firstLi = ul.firstElementChild;
  var textNode = document.createTextNode("텍스트 노드 변경");

   firstLi.replaceChild(textNode, firstLi.firstChild);
});
```

- closest(): 상위로 올라가면서 가장 가까운 Element를 반환
  - ex)

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width">
  <title>부트스트랩</title>
</head>
<body>
  <div>
    <ul>
      <li id="list1">1. 웹 프로그래밍 기초</li>
      <li id="list2">2. DB연결 웹 애플리케이션</li>
      <li id="list3">3. 웹 애플리케이션 개발 1/4</li>
    </ul>
  </div>
</body>
</html>
```

```javascript
var firstLi = document.getElementById("list1");

console.log(firstLi.closest("div").nodeName);

console output:
"DIV"
```

### 4. HTML을 문자열로 처리해주는 DOM 속성 / Method

- innerText: 해당 Node와 모든 Descendant의 text 내용을 설정하거나 반환
  - ex)

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width">
  <title>부트스트랩</title>
</head>
<body>
  <div>
    <ul>
      <li id="list1">1. 웹 프로그래밍 기초</li>
      <li id="list2">2. DB연결 웹 애플리케이션</li>
      <li id="list3">3. 웹 애플리케이션 개발 1/4</li>
    </ul>
  </div>
  <button id="copyTextButton">두번째 리스트 내용을 첫번째 리스트 내용으로 변경</button>
</body>
</html>
```

```javascript
var copyTextButton = document.getElementById("copyTextButton");

copyTextButton.addEventListener("click", function() {
  var list1Text = document.getElementById("list1").innerText;
  
  document.getElementById("list2").innerText = list1Text;
});
```

- innerHTML: 해당 Element 내부의 html을 설정하거나 반환
  - ex)

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width">
  <title>부트스트랩</title>
</head>
<body>
  <div>
    <ul>
      <li id="list1">1. 웹 프로그래밍 기초</li>
      <li id="list2">2. DB연결 웹 애플리케이션</li>
      <li id="list3">3. 웹 애플리케이션 개발 1/4</li>
    </ul>
  </div>
  <button id="copyListButton">첫번째 리스트만 남기고 제거</button>
</body>
</html>
```

```javascript
var copyListButton = document.getElementById("copyListButton");

copyListButton.addEventListener("click", function() {
  var ul = document.querySelector("ul");
  var list1Html = document.getElementById("list1").innerHTML;
  
  ul.innerHTML = list1Html;
});
```

- insertAdjacentHTML(): HTML로 텍스트를 지정된 위치에 삽입
  - ex)

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width">
  <title>부트스트랩</title>
</head>
<body>
  <div>
    <ul>
      <li id="list1">1. 웹 프로그래밍 기초</li>
      <li id="list2">2. DB연결 웹 애플리케이션</li>
      <li id="list3">3. 웹 애플리케이션 개발 1/4</li>
    </ul>
  </div>
  <button id="addListButton">리스트 추가</button>
</body>
</html>
```

```javascript
var addListButton = document.getElementById("addListButton");

addListButton.addEventListener("click", function() {
  var list3 = document.getElementById("list3");
  
  list3.insertAdjacentHTML("afterend", "<li>4. 웹애플리케이션 개발 2/4</li>");
});
```

### 5. 실습

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width">
  <title>JS Bin</title>
</head>
<body>
  <h1>selector test</h1>
  
  <ul>
    <li>apple</li>
    <li>orange</li>
    <li>banana</li>
    <li>grape</li>
    <li>strawberry</li>
  </ul>

</body>
</html>
```

- 지금 나온 DOM API를 사용해서, strawberry 아래에 새로운 과일을 하나 더 추가하시오.

  추가 된 이후에는 다시 삭제하시오.

```javascript
var ul = document.querySelector("ul");
var li = document.createElement("li");
var pineapple = document.createTextNode("pineapple")

li.appendChild(pineapple);
ul.appendChild(li);
ul.removeChild(li);
```

- insertBefore메서드를 사용해서, orange와  banana 사이에 새로운 과일을 추가하시오.

```javascript
var ul = document.querySelector("ul")
var banana = document.querySelector("li:nth-child(3)")
var li = document.createElement("li");
var pineapple = document.createTextNode("pineapple")

li.appendChild(pineapple);
ul.insertBefore(li,banana);
```

- 바로 위의 내용을 insertAdjacentHTML메서드를  사용해서, orange와 banana 사이에 새로운 과일을 추가하시오.

```javascript
var banana = document.querySelector("li:nth-child(3)")

banana.insertAdjacentHTML("beforebegin", "<li>pineapple</li>");
```

- apple을 grape 와 strawberry 사이로 옮기시오.

```javascript
var ul = document.querySelector("ul");
var apple = document.querySelector("li:nth-child(1)");
var strawberry = document.querySelector("li:nth-child(5)")

ul.insertBefore(apple, strawberry);
```

- class 가 'red'인 노드만 삭제하시오.

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width">
  <title>JS Bin</title>
</head>
<body>
  <h1>selector test</h1>
  
  <ul>
    <li class="red">apple</li>
    <li class="red">orange</li>
    <li>banana</li>
    <li>grape</li>
    <li>strawberry</li>
  </ul>

</body>
</html>
```

```javascript
document.querySelectorAll("li.red")
  .forEach(function(element) {
  	element.remove();
});
```



- section 태그의 자손 중에 blue라는 클래스를 가지고 있는 노드가 있다면, 그 하위에 있는 h2 노드를 삭제하시오.

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width">
  <title>JS Bin</title>
</head>
<body>
  <h1>selector test</h1>
  
    
<section>
  <h2> red section </h2>
  <ul>
    <li class="red">apple</li>
    <li class="red">orange</li>
    <li>banana</li>
    <li>grape</li>
    <li>strawberry</li>
  </ul>
</section>
  
  <Br>
  
<section>
  <h2> blue section </h2>
  <ul>
    <li class="green blue">apple</li>
    <li class="red">orange</li>
    <li>banana</li>
    <li>grape</li>
    <li>strawberry</li>
  </ul>
</section>

</body>
</html>
```

```javascript
var blues = document.querySelectorAll("section .blue");

blues.forEach(function(element) {
  element.closest("section").querySelector("h2").remove();
});
```



### 6. Polyfill

- 오래된 브라우저나 특정 기능이 지원되지 않는 브라우저를 위해 사용할 수 있는 코드
- 참고링크: <https://hacks.mozilla.org/2014/11/an-easier-way-of-using-polyfills/>



원본 강의: https://www.edwith.org/boostcourse-web