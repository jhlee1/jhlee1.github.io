---
layout: post
title: "Inflearn Vue nodebird 강의노트"
date: 2020-03-04 11:33:54 +09:00
tags: ["vue", "javascript", "nodebird"]
categories: [vue]
---
## I. Setup

1. 빈 프로젝트에 front와 back폴더를 나눠서 설정
   1. 나중에 backend도 추가할꺼임
2. 해당 폴더에 terminal로 이동해서 명령어를 입력 후 패키지명만 입력하고 나머지는 그냥 enter

```bash
$ npm init

package name: (front) vue-nodebird-front
version: (1.0.0) 
description: 
entry point: (index.js) 
test command: 
git repository: 
keywords: 
author: 
license: (ISC) 
```

2. vue랑 nuxt 설치하기

```bash
$ npm i vue nuxt
```

3. pages라는 폴더를 만들고 그 안에 `index.vue`, `profile.vue`, `signup.vue` 파일을 넣는다
   1. vue만 쓰면 상관 없는데 nuxt를 사용하기 때문에 파일 이름이 중요하다
   2. pages 아래 folder명과 파일명으로 라우팅이됨
   3. `pages/user/profile.vue` => `localhost:3000/user/profile`
4. package.json에서 개발모드로 설정

```json
"scripts" : {
	"dev" : "nuxt"

```

5. 개발모드로 실행

```bash
npm run dev
```

6. vuetify 설치

```bash
npm i vuetify @nuxtjs/vuetify
```

7. eslint설치하기
   1. -D는 개발용으로만 쓰겠다는 뜻

```bash
npm i -D eslint eslint-plugin-vue
```

8. nuxt 폴더는 빌드된 결과물이 다 들어있음 (Java의 target폴더 느낌)
   1. `.gitignore`에 node_modules랑 같이 추가

```
.nuxt
node_modules
```

## II. Vuex

- 2가지 모드가 있음
  - Classic mode
    - 기존 export default로 코딩하던 방식처럼 mutation과 state가 하나에 들어있음
  - Module mode
    - mutation과 state가 하나에 들어있음
    - 파일을 분리하기 쉬움
    - 유저 관련 module, 상품관련 module 분리할 수 있음 classic mode는 하나의 state에 다 때려넣음





