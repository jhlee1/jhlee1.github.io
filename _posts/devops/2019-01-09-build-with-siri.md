---
layout: post
title: "\"시리야 빌드해줘\"로 Jenkins에서 빌드하기"
date: 2020-01-09 10:27:29 +0900
tags: ["jenkins", "siri"]
categories: [devops]
image: shorcut-jenkins-iphone.jpeg
---

## I. Jenkins Plugin 설치

- Manage Jenkins > Manage Plugins으로 들어가기
- Available 클릭 후 Build Authorization Token Root Plugin 플러그인 설치

## II. Jenkins Job에 토큰등록하기

- Siri를 통해 빌드할 Job의 Configure에 들어가기
- Build Triggers에서 Trigger builds remotely (e.g., from scripts) 체크하기
- 체크하면 Authentication Token을 입력하는 부분이 생기는데 거기에 Token을 만들어서 입력해주면됨 (이 토큰은 밑에서 쓰인다)

## III. Shortcuts(단축어) 설정

- iPhone 혹은 iPad에서 Shortcuts(단축어) 앱 실행
- Create Shortcut(단축어 생성) 클릭 후 동작추가
- 동작 `URL` 추가 후 `http://젠킨스_주소/buildByToken/build\?job=Job_이름&token=해당_Job의_Authentication_Token` 입력
- 그 밑에 `URL 콘텐츠 가져오기` 추가 후 Method(메소드)를 POST로 설정
- 오른쪽 아래 Play 버튼으로 테스트해서 제대로 동작하는지 확인하기
- 단축어명은 알아서 정하기 (나의 경우엔 `빌드`로 정함)

![shorcut-jenkins-iphone](/static/img/posts/devops/shorcut-jenkins-iphone.jpeg)
## IV. 실행하기

- "시리야 `빌드`(단축어 명) 해줘" 라고하면 젠킨스 Job이 실행됨





