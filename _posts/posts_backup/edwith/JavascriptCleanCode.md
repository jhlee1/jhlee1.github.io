---
title: "Javascript Clean Code"
date: 2019-05-11T22:48:54+09:00
archives: "2019"
tags: ["javascript", "edwith", "webProgramming"]
author: Joohan Lee
---

## Clean Code

### 1. 개념

- 읽기 좋은 코드, 가독성 있는 코드
- 소프트 웨어 특성상 한번 만들어진 후 계속 유지보수가 필요함 -> 다른 사람이 이해할 수 있는 코드를 작성해야함
- [Airbnb의 Javascript Coding Convention](https://github.com/airbnb/javascript)을 참고



### 2. 이름 (코딩컨벤션)

- 함수는 목적에 맞게 이름이 지어져 있는가?
- 함수 안의 내용은 이름에 어울리게 하나의 로직을 담고 있는가?
- 함수는 동사 + 명사이며 함수의 의도를 충분히 반영하고 있는가?
- 함수는 카멜표기법 또는 _를 중간에 사용했는가?
- 변수는 명사이며 의미 있는 이름을 지었는가?

ex) 잘못된 함수명 (동사가 없어서 무슨 일을 하는지 알 수 없음)

```javascript
function double(value) {
    return value * 2;
}
```

ex) 올바른 함수명 (좋은 예는 아니지만 위의 함수보다 구체적임. 이름이 조금 길어져도 구체적으로 쓰는게 낫다)

```javascript
function getDoubleValue(value) {
    return value * 2;
}

function addSemiColonStr(value) {
    return value + ";";
}
```



### 3. 의도가 드러난 구현패턴

- 코드를 보고 의도를 파악할 수 있어야함

```javascript
var value *= 19.2;
```

- 19.2가 무엇을 의미하는지 알 수가 없다 > 대신 변수로 저장하고, 변수에 적절한 이름사용

```javascript
var padding = 19.2;
var value *= padding;
```



### 4. 지역(local)변수로 선언하면 될 것을 전역(global)변수로 선언하지 말자

- 함수 내에서만 사용할 것은 지역 변수로 선언할 것
- 불필요한 전역 변수를 최소화 해야 코드가 길어졌을 때 수정이 편하다
  - 아래의 상황에서 코드가 길어지면 name을 어디서 가져오는지 찾기 힘듬
  - 중간에 name의 값이 변경되면 결과가 바뀔 수 있다.

```javascript
var name = 'John';

function print() {
	return data;
}

function exec() {
	var data = "world";
	name = name + " ";
	
	print(name + data);
}
```



### 5. 빨리 return해서 if 없애기

- 아래의 코드에서 if 중첩 없애기

```javascript
function foo(john, smith) {
    if(pobi) {
        if (crong) {
            return true;
        }
    }
}
```

- pobi가 아니면 미리 return하기

```javascript
function foo(john, smith) {
    if(!pobi) return;
    if (crong) {
        return true;
    }
}
```



### 6. 정적 분석도구 사용

- eslint

  - 잠재적인 문제와 anit-pattern을 알려줌

  - IDE에서 plugin 추가



