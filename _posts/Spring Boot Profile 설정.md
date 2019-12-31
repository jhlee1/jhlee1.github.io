# Spring Boot Profile 설정

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



IV. 실행시 프로파일 설정

```
$ java -jar ... -D spring.profiles.active=dev
```

