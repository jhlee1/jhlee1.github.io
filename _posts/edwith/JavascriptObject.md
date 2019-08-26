---
title: "JavascriptObject"
date: 2019-02-27T22:19:33+09:00
archives: "2019"
tags: ["javascript", "edwith", "webProgramming"]
author: Joohan Lee
---

## Javascript Object

### 1. 객체

- key, value의 자료구조
- {}로 자료를 표현하며, JSON형식이랑 같다.

### 2. 선언

```javascript
var obj = {name: "John Doe", age: 20, addition: [{name: "another one", age: 99}]}

console.log(obj.name); // John Doe
console.log(obj["name"]); // John Doe
console.log(obj[name]); // Error : name이라는 변수를 찾기 때문에
console.log(obj.addition[0].name); // another one

```

### 3. 객체 탐색

- For-in loop
  - for-in은  key값을 탐색하기 위한 것이므로 Array(배열)가 아니라 Object(객체)에 사용한다

```
var obj = {name: "John Doe", age: 20, addition: [{name: "another one", age: 99}]}

for(key in obj) {
    console.log(myFriend[key]);
}
```

- Object.keys()

```
var obj = {name: "John Doe", age: 20, addition: [{name: "another one", age: 99}]}

console.log(Object.keys(obj)); // ['name', 'age', 'addition']
```



### 4. 실습

1. 숫자 타입으로만 구성된 요소를 뽑아 배열을 만들기

```json
const data = {
    "debug": "on",
    "window": {
        "title": "Sample Konfabulator Widget",
        "name": "main_window",
        "width": 500,
        "height": 500
    },
    "image": { 
        "src": "Images/Sun.png",
        "name": "sun1",
        "hOffset": 250,
        "vOffset": 250,
        "alignment": "center"
    },
    "text": {
        "data": "Click Here",
        "size": 36,
        "style": "bold",
        "name": "text1",
        "hOffset": 250,
        "vOffset": 100,
        "alignment": "center",
        "onMouseUp": "sun1.opacity = (sun1.opacity / 100) * 90;"
    }
}
```

- 실행결과

```json
["width", "height", "hOffset", "vOffset", "size", "hOffset", "vOffset"]
```

2. type이 sk인, name으로 구성된 배열만 출력

```json
[{
	"id": 1,
	"name": "Yong",
	"phone": "010-0000-0000",
	"type": "sk",
	"childnode": [{
		"id": 11,
		"name": "echo",
		"phone": "010-0000-1111",
		"type": "kt",
		"childnode": [{
				"id": 115,
				"name": "hary",
				"phone": "211-1111-0000",
				"type": "sk",
				"childnode": [{
					"id": 1159,
					"name": "pobi",
					"phone": "010-444-000",
					"type": "kt",
					"childnode": [{
							"id": 11592,
							"name": "cherry",
							"phone": "111-222-0000",
							"type": "lg",
							"childnode": []
						},
						{
							"id": 11595,
							"name": "solvin",
							"phone": "010-000-3333",
							"type": "sk",
							"childnode": []
						}
					]
				}]
			},
			{
				"id": 116,
				"name": "kim",
				"phone": "444-111-0200",
				"type": "kt",
				"childnode": [{
					"id": 1168,
					"name": "hani",
					"phone": "010-222-0000",
					"type": "sk",
					"childnode": [{
						"id": 11689,
						"name": "ho",
						"phone": "010-000-0000",
						"type": "kt",
						"childnode": [{
								"id": 116890,
								"name": "wonsuk",
								"phone": "010-000-0000",
								"type": "kt",
								"childnode": []
							},
							{
								"id": 1168901,
								"name": "chulsu",
								"phone": "010-0000-0000",
								"type": "sk",
								"childnode": []
							}
						]
					}]
				}]
			},
			{
				"id": 117,
				"name": "hong",
				"phone": "010-0000-0000",
				"type": "lg",
				"childnode": []
			}
		]
	}]
}]
```

- 실행 결과

```javascript
["Yong", "hary", "solvin", "hani", "chulsu"]
```



원본 강의: https://www.edwith.org/boostcourse-web