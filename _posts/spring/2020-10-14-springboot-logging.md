---
layout: post
title: "Spring boot jdbc logging 편리하게 하기"
date: 2020-10-14 17:20:00 +0900
tags: ["spring", "java", "jdbc", "log", "logging"]
categories: [spring]
---

## I. JDBC SQL문 깔끔하게 프린팅되도록 만들기

- Spring boot에서 JDBC를 이용한 서비스를 개발하다보면 SQL을 확인해야할 때가 있다
- spring-data-jdbc의 경우 로깅을 제대로 지원하지 않기 때문에 방법을 찾다가 좋은 라이브러리를 발견
- 기존 hibernate로 로깅하는 방식의 경우 보내는 쿼리가 깔끔하게 출력되지 않는다

## II. Dependency 추가하기

```groovy
    implementation 'com.integralblue:log4jdbc-spring-boot-starter:2.0.0'
```

## III. application.yml에 설정 추가하기

```yaml
jdbc:
    resultsettable: info
    sqltiming: info
    sqlonly: fatal
    audit: fatal
    resultset: fatal
    connection: fatal
```

## IV. 참고 링크

- [Stack overflow](https://stackoverflow.com/questions/45346905/how-to-log-sql-queries-their-parameters-and-results-with-log4jdbc-in-spring-boo)
- [Github](https://github.com/candrews/log4jdbc-spring-boot-starter)