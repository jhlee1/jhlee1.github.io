---
title: "LayeredArchitectureImplementation"
date: 2019-05-05T18:01:53+09:00
archives: "2019"
tags: ["spring", "edwith", "webprogramming"]
author: Joohan Lee
---

## Layered Architecture 실습

### 1. 방명록 만들기

- Spring JDBC를 이용한 DAO 작성
- Controller + Service + Dao
- Transaction 처리
- Spring MVC에서 form 값 입력받기
- Spring MVC에서 redirect하기
- Controller에서 jsp에게 전달한 값을 JSTL과 EL을 이용해 출력하기
- 설정파일
  - Web Layer 설정 파일: web.xml, WebMvcContextConfiguration.java
  - Business, Repository Layer 설정 파일: ApplicationConfig.java, DbConfig.java



### 2. 요구 사항

- 방명록 정보는 guestbook Table에 저장
  - id는 자동 입력
  - id, 이름, 내용, 등록일을 저장
- http://localhost:8080/guestbook/을 요청하면 자동으로 /guestbook/list로 redirect
  - 방명록이 없으면 건수는 0이 나오고 아래에 방명록을 입력하는 form이 보여진다.
  - 이름과 내용을 입력하고, 등록 버튼을 누르면 /guestbook/write URL로 입력한 값을 전달하여 저장
  - 값이 저장된 이후에는 /guestbook/list로 redirect됨 
  - 입력한 한건의 정보가 보여진다.
- 방명록의 내용과 form사이의 숫자는 방명록 페이지 링크. 1페이지당 5건
  - 1페이지를 누르면 /guestbook/list?start=0
  - 2페이지를 누르면 /guestbook/list?start=5
  - /guestbook/list는 /guestbook/list?start=0과 결과가 같다
- 방명록에 글을 쓰거나 방명록에 글을 삭제할 때는 Log Table에 클라이언트의 ip주소, 등록(삭제) 시간, 등록/삭제(method column) 정보를 DB에 저장
  - 사용하는 table은 log
  - id는 자동으로 입력



실습 소스코드:  <https://github.com/jhlee1/edwith_guestbook>

원본 강의: https://www.edwith.org/boostcourse-web