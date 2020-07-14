---
layout: post
title: "Spring Boot Profile 설정"
date: 2020-04-13 23:00:00 +0900
tags: ["spring", "java"]
categories: [spring]
---

## I. 표현의 한계로 .properties보다 .yml을 더 많이 사용하는 추세

## II. Application.yml에서 Profile 분리하기

- `---` 으로 분리

```yaml
server:
	port: 80
---
spring:
	profiles: dev
server:
	port: 8080
---
spring:
	profiles: stage
server:
	port: 8081
---
spring:
	profiles: prod
server:
	port: 8082
```

## III. 파일별 분리

- applicaiton-{profile}.yml
- 공통 정보는 application.yml에 넣은 후 dev, stage, prod으로 실행시 해당 profile이 먼저 참조되고 없으면 application.yml값을 참조함

## IV. 다른 Profile 포함시키기

- `application-db.yml, application-dev.yml` 이런식으로 파일 여러개로 분리해서 저장할 경우 아래 문장을 추가하면 해당 profile에 포함된다.

```
spring.profiles.include: db,dev
```



## IV. 실행시 프로파일 설정

```bash
$ java -jar ... -D spring.profiles.active=dev
```
