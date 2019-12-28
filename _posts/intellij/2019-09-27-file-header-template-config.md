---
layout: post
title:  "Intellij File Template을 이용해서 파일 헤더에 작성자 추가하기"
date:   2019-09-27 12:35:28 +0900
categories: intellij
tags: ["velocity", "intellij"]
---

## 1. File Header 들어가기

- Preferences -> Editor -> File And Code Templates -> Includes -> File Header

## 2. Header에 넣고 싶은 부분 넣기

- Velocity 문법으로 표현하면됨

```velocity
/**
* Created by John Doe on ${DATE}
*/

```

- 결과

```velocity
/**
* Created by John Doe on 2019-09-27
*/
```

## 3. Project 이름 별로 다른 Header 사용하기

- 내 개인 프로젝트와 회사 프로젝트를 같은 머신 (같은 Intellij)에서 처리할 경우 프로젝트별 설정이 불가능
- 따라서, 프로젝트 이름을 기준으로 header에 들어갈 내용을 분리해줘야함 
  - Regex를 이용해서 프로젝트 명이 
    - **company**혹은 **com**로 시작할 때
    - **toy**혹은 **personal**로 시작할 때
    - 그 외의 경우를 분리

```velocity
#set($projectName = ${PROJECT_NAME})
#if($projectName.matches("^(company|com)"))
/**
* Created by Joohan Lee on ${DATE} at My Current Company
* 
*/
#elseif($projectName.matches("^(toy|personal)"))
/**
* Created by Joohan Lee on ${DATE} at your blog address
*/
#else
/**
* Created by Joohan Lee on ${DATE}
*/
#end
```

## 4. Template에서 사용할 수 있는 변수들 정보
- (https://www.jetbrains.com/help/idea/file-template-variables.html)[https://www.jetbrains.com/help/idea/file-template-variables.html]