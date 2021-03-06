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

## III. Node.js 설정

1. Mysql 설치 5.7.x 추천
2. npm으로 프로젝트 설정

``` bash
$ npm init
$ npm i express


```

3. package.json 설정
   1. "main" 부분 수정과 script에 "dev" 추가

```
{
  "name": "vue-nodebird-back",
  "version": "1.0.0",
  "description": "",
  "main": "app.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "node app.js"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "express": "^4.17.1",
    "mysql2": "^2.1.0",
    "sequelize": "^5.21.5"
  }
}

```

4. app.js 파일 추가 후 실행해서 잘 작동하는지 확인하기

```javascript
const express = require('express');

const app = express();

app.get('/', (req, res) => {
   res.status(200).send('안녕 백엔드');
});

app.listen(3085, () => {
    console.log(`백엔드  서버 ${3085}번 포트에서 작동중.`);
})

```

5. Dependency 추가
   1. `npx`:  명령어로 쓸 수 있도록 해준다
   2. EX) 기존 npm i -g sequelize-cli로 설치해서 `sequelize xxx` 이런식으로 많이 썼는데, 이 경우 package.json에 해당 dependency가 추가되지 않는 문제가 있어서 local로 설치하고 npx를 이용해서 명령어 실행

```bash
$ npm i sequelize mysql2
$ npm i -D sequelize-cli
$ npx sequelize init
$ npm i -D nodemon
```

6. nodemon 사용
   1. package.json의 script 수정

```
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "nodemon app.js"
  },
```

7. sequelize로 db 생성

```bash
$ npx sequelize db:create
```
