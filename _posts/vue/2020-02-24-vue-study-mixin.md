layout: post
title: "OGQ Vue 스터디 정리 노트 - Mixin"
date: 2020-02-24 11:33:54 +09:00
tags: ["vue", "javascript", "mixin"]
categories: [vue]



## I. 개념

- 2개의 vue component내에서 중복되는 methods, props, data등이 존재하는 경우 그 부분을 한 곳으로 뺄 때 사용
  - component의 전처리 개념으로 사용
- component 보다 이전에 실행됨
  - Spring AOP Before advice 또는 JUnit 5의 BeforeEach와 비슷한 느낌

## II. 기본 예제

```javascript
var exampleMixin = {
		created: function () {
		console.log('From mixin.')
	},
}
    
var Component = Vue.extend({
		mixins: [exampleMixin],
		created: () => {
		console.log("From component.")
	}
})

var component = new Component()

// => "From mixin."
// => "From component."
```

## III. 동일한 data, method가 존재할 경우 component가 우선순위를 가진다

```javascript
    var exampleMixin2 = {
        data: function () {
            return {
                message: 'From mixin',
                another: 'Another from mixin'
            }
        }
    }

    var Component2 = Vue.extend({
        mixins: [exampleMixin2],
        data: function () {
            return {
                message: 'From component',
                anotherComp: 'Another from component'
            }
        },
        created: function () {
            console.log(this.message);
            console.log(this.another);
            console.log(this.anotherComp);
        }
    })

    var component2 = new Component2();
    // From component
    // Another from mixin
    // Another from component

```

## IV. 전역 Mixin 설정

- 모든 Vue instance와 component들에 적용됨
  - 주의해서 사용해야한다

```javascript
Vue.mixin({
    created: function () {
            console.log('global mixin!');
    }
})

var Component3 = Vue.extend({
    created: function () {
        console.log('component created!');
    }
})

var component3 = new Component3();

// => "global mixin!"
// => "component created!"
```



## V. Merge 옵션설정

- 충돌이 발생할 경우 어떤식으로 처리할지 정의

```
Vue.config.optionMergeStrategies.myOption = function (toVal, fromVal) {
  // return 병합된 값
}
```























V. https://www.vuemastery.com/courses/next-level-vue/mixins/