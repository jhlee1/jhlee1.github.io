---
layout: post
title: "Vue Js Crash Course 2019 따라하기"
date: 2019-05-12 11:33:54 +09:00
tags: ["vue", "javascript"]
categories: [vue]
---
## 1. Setup
- npm을 사용하기 위해 Node js 설치
  - https://nodejs.org/ko/에서 받아서 설치하면 됨
- vue cli 설치 (global로 설정)
  - `npm install -g @vue/cli`
- 잘 설치 되었는지 확인 (버전 확인)
  - `vue --version`
- 에디터 설치 (VSCode, Atom 등등)
  - 영상에선 VSCode를 사용하니 VSCode 설치 <https://code.visualstudio.com/
  - 에디터의 Plugin으로 Vueter도 같이 설치해주자.

## 2. 프로젝트 생성하기

- CLI(Command Line Interface)로 프로젝트 생성하기 (`vue create 프로젝트명`)
  - workspace로 이동
  - `vue create test`
  - preset은 default로 하기 (babel, eslint)
  - 잘 생성된지 검사
    - 생성된 프로젝트 디렉토리로 이동 후 `npm run serve`
    - `http://localhost:8080`에 접속
- Vue UI로 프로젝트 생성
  - workspace로 이동 후 `vue ui` 입력
  - 생성된 브라우저창에서 create 클릭 후 생성
  - 왼쪽편에 plugins, dependencies, configurations 등 보기 편함

## 3. 프로젝트 살펴보기

- package.json
  - dependency를 모아놓은 곳 (pom.xml같은 존재)
- index.html
  - 유일하게 불러오는 HTML 파일
  - Vue는 SPA이기 때문에 하나의 HTML 파일만 불러온다
  - `<div id="app"></div>` - 우리가 추가할 부분의 placeholder 역할

- main.js
  - component의 엔트리 포인트
  - App.vue instance를 생성 후 index.html의 `<div id='app'></div>`에 넣어줌
- index.html -> main.js -> App.vue 순서로 참조

- App.vue
  - 메인 Component
  - 현재 HelloWorld component를 참조하고 있음
  - 페이지 내에서 모두 적용할 style 선언

## 4. 시작하기

- To do list 만들기
- 기본적으로 Vue Component는 template, script, style을 가지고 있음
  - `<template>`안에는 하나의 `<div>`만 존재해야함
  - `<script>` 비지니스 로직 (js 코드)를 가지고 있음
    - props -  받을 parameter를 정의
    - data - instance를 return하는 function
  - `<style>` CSS 정의 `<style scope>` 해당 component에서만 적용됨
- HelloWorld componet 지우고 todo 더미 데이터 넣어주기
  - HelloWorld.vue 파일 삭제 후
  - App.vue에서 HelloWorld를 참조하던 부분을 삭제
  - 스타일 변경

```javascript
<template>
  <div id="app">
    {{todos}}
  </div>
</template>

<script>

export default {
  name: 'app',
  components: {
  },
  data() {
    return {
      todos: {
        
      }
    }
  }
}
</script>

<style>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: Arial, Helvetica, sans-serif;
  line-height: 1.4;
}
</style>

```

- Todos.vue 추가
- TodoItem.vue 추가

- components/layout/Header.vue 추가



## 5. jsonplaceholder로 http 통신 추가하기

- 이미 만들어진 todo list 가져오기 <https://jsonplaceholder.typicode.com/todos> 사용
- App.vue와 AddTodo에서 변경되는 내용
  - App.vue의 method 내용 변경
  - AddTodo에서 uuid를 더이상 사용하지 않음

## 6. Vue Router 설치하기

- vue-router plugin을 설치하면 App.vue에 덮어쓰기 하기 때문에 미리 복사해서 저장해놓자
- `vue ui`를 이용해서 설치하자
  - Plugins에 들어가서 add vue-router 눌러서 설치하면 됨
- 아까 복사해놓은 App.vue의 내용을 view/Home.vue에 넣고 현재 디렉토리가 변경되었으므로 import from에서 `./components/...`를 `../components/...`로 변경
- nav관련 내용을 App.vue에서 Header로 옮겨주기
- Header관련 내용을 Home.vue에서 App.vue로 옮기기



## 7. Deploy하기위해 build하기

- 두가지 방식이 있음
  - cmd에서
    - npm run build로 실행하면 dist에 생성됨
  - vue ui에서
    - tasks에 build실행
    - configurations에서 위치 변경할 수 있음



