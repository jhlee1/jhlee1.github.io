---
title: "RESTAPI"
date: 2019-02-26T23:29:11+09:00
archives: "2019"
tags: ["RESTAPI", "edwith", "webProgramming"]
author: Joohan Lee
---

## REST API



### 1. REST API 조건

- REST API라 불리기 위해선 아래의 조건들이 필요하다. 대부분의 조건은 만족하기 쉽지만 웹페이지가 아닌 API상에서 **uniform interface**의 하위 조건 중 일부를 만족하기 어려움
- client-server
- stateless
- cache
- uniform interface
- layered system
- code-on-demand (optional)
- 위의 조건중 **uniform interface**의 조건
  - 리소스가 URI로 식별되야 한다.
  - 리소스를 생성,수정,추가하고자 할 때 HTTP메시지에 표현을 해서 전송해야 한다.
  - 메시지는 스스로 설명할 수 있어야 한다. (Self-descriptive message)
  - 애플리케이션의 상태는 Hyperlink를 이용해 전이되어야 한다.(HATEOAS)
- 이 중 JSON을 이용해 HTTP통신을 하는 API 상에선 Self-descriptive message와 HATEOAS을 만족 시키기 위해선 많은 시간과 노력이 필요
- REST API가 되기 위한 모든 조건을 만족시키기 위해선 많은 시간과 노력이 필요하기 때문에 대부분의 웹사이트에서 지켜지지 않고 있다. 이러한 경우 아래와 같은 방식을 따른다.
  - REST의 모든 것을 제공하지 않으면서 REST API라고 부르기
  - REST의 모든 것을 제공하지 않고 Web API 혹은 HTTP API라고 부르기

### 2. Web API (HTTP API) 디자인 가이드

- URI는 정보의 자원을 표현해야 한다.
- 자원에 대한 행위는 HTTP Method(GET, POST, PUT, DELETE)로 표현됨

| Method | 역할                          |
| ------ | ----------------------------- |
| POST   | 생성                          |
| GET    | 조회하고 자세한 정보를 가져옴 |
| PUT    | 수정                          |
| DELETE | 삭제                          |

- ex) GET /members
  - 멤버에 대한 모든 정보를 달라는 요청
-  ex) GET /members/delete/1 (X)
  - GET은 정보를 요청할 떄 사용되어야함. 위와 같이 동사로 삭제를 표현하면 안됨
  - DELETE /members/1  (O) 처럼 DELETE method를 이용해야함
  - GET /members/1                   (o)
  - GET /members/get/1             (x)
  - GET /members/add                 (x)
  - POST /members                       (o)
  - GET /members/update/1        (x)
  - PUT /members/1                     (o)
  - GET /members/del/1               (x)
  - DELETE /members/1               (o)
- Slash(/) 는 계층을 나타낼 때 사용
  - URI 마지막 문자로 슬래시 구분자(/)를 포함하지 않는다.
  - 하이픈(-)은 URI가독성을 높일 때 사용합니다.
  - 언더바(_)는 사용하지 않습니다.
  - URI경로는 소문자만 사용합니다.
  - RFC 3986(URI 문법 형식)은 URI스키마와 호스트를 제외하고는 대소문자를 구별합니다.
  - 파일 확장자는 URI에 포함하지 않습니다.
  - Accept Header를 사용합니다.
  - ex) http://domain/houses/apartments
    - houses 중에 apartments들을 요청
  - ex) http://domain/departments/1/employees
    - department 1에 있는 employee들을 요청
- 상태 코드

| 상태 | 상태코드 | 설명                                                         |
| ----- | -------- | ------------------------------------------------------------ |
| 정상 | 200      | 요청이 정상적으로 수행됨                                     |
| 정상 | 201      | POST를 이용해서 생성 요청시 정상적으로 생성됨                |
| 클라이언트 오류 | 400      | 클라이언트의 요청이 부적절할 경우                            |
| 클라이언트 오류 | 401      | 인증되지 않은 상태에서 보호된 리소스를 요청한 경우 (로그인이 필요한 정보를 로그인 하지 않은 유저가 요청했을 경우) |
| 클라이언트 오류 | 403      | 유저 인증 상태와 관계 없이 응답하고 싶지 않은 리소스를 클라이언트에서 요청한 경우 (403 자체가 해당 리소스가 존재한다는 것을 알려주는 것이므로 400이나 404를 사용할 것을 권고) |
| 클라이언트 오류 | 405      | 요청한 리소스에서는 사용 불가능한 Method를 이용한 경우       |
| 서버 오류 | 301      | 요청한 리소스의 URI가 변경된 경우 (Location Header에 변경된 URI를 같이 적어줘야함) |
| 서버 오류 | 500      | 서버에 문제가 있을 경우                                      |





### 참고 링크

- https://www.edwith.org/boostcourse-web/lecture/16740/
- https://www.youtube.com/watch?v=RP_f5dMoHFc
