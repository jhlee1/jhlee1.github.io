---
title: "SQL Basic"
date: 2019-02-16T21:45:07+09:00
archives: "2019"
tags: ["edwith", "SQL", "webProgramming", "database"]
author: Joohan Lee
---

## 1. SQL이란

- SQL은 데이터를 쉽게 검색하고 추가, 삭제, 수정 같은 조작을 할 수 있도록 만들어진 언어
- RDBMS에서 데이터를 조작, 쿼리하는 표준 수단
- 키워드는 대소문자를 구분하지 않음 (아래의 쿼리는 모두 같다)
  - mysql> SELECT VERSION(), CURRENT_DATE;
  - mysql> select version(), current_date;
  - mysql> SeLeCT vErsiOn(), CUrrENT_DATE;
- QUERY를 계산식으로 쓸 수 있다.
  - mysql> SELECT SIN(PI() / 4),  (4+1) * 5;

```
+--------------------+-----------+
| SIN(PI() / 4)      | (4+1) * 5 |
+--------------------+-----------+
| 0.7071067811865476 |        25 |
+--------------------+-----------+
1 row in set (0.00 sec)
```

- 여러 문장을 한 줄에 연속 붙여서 실행 가능 (;만 정확히 붙여주기)
  - mysql> SELECT version(); SELECT NOW();

```
+-----------+
| VERSION() |
+-----------+
| 8.0.15    |
+-----------+
1 row in set (0.00 sec)

+---------------------+
| NOW()               |
+---------------------+
| 2019-02-16 22:26:45 |
+---------------------+
1 row in set (0.00 sec)
```

- 하나의 SQL을 여러 줄로 입력 가능
  - mysql> SELECT
  - ​         -> USER()
  - ​         -> ,
  - ​         -> CURRENT_DATE;

```
+-----------------------+--------------+
| USER()                | CURRENT_DATE |
+-----------------------+--------------+
| user@localhost | 2019-02-16   |
+-----------------------+--------------+
1 row in set (0.00 sec)

```

- 입력 도중  취소 가능. 중간에 취소해야되는 경우 \c를 붙여주면 됨
  - mysql> SELECT
  - ​         -> USER()
  - ​         -> \c

## 2. 분류

- DML(Data Manipulation Language): 데이터를 조작하기 위해 사용.
  - ex) INSERT, UPDATE, DELETE, SELECT 등
- DDL(Data Definition Language): DB의 Schema를 정의 및 조작
  - ex) CREATE, DROP, ALTER 등
- DCL(Data Control Language): 권한 관리, 데이터의 보안, 무결성 등을 정의
  - GRANT, REVOKE 등

### 3. Database 생성, 유저생성 후 권한주기

- mysql -uroot -p
  - MySQL 관리자 계정인 root로 DBMS에 접속
  - window에선 설치시 입력했던 암호, Mac은 그냥 엔터치면됨
- mysql> create database *DBname*:
  - *DBname*으로 데이터베이스를 생성
- CREATE USER '**username**'@'%' IDENTIFIED BY '**password**';
  - 유저 생성
- GRANT ALL PRIVILEGES ON **DBname**.* to **username**@'%';
  또는 GRANT ALL PRIVILEGES ON **DBname**.* to **username**@'localhost';
  - 위의 명령어는 유저에게 권한을 주는 것
  - **DBname**뒤에 *는 모든 권한을 준다는 뜻
  - @'%'는 모든 client에서 접근이 가능하다는 뜻
  - % 대신 localhost를 사용하면 현재 MySQL서버가 설치된 컴퓨터에서만 접근할 수 있다는 뜻
- FLUSH PRIVILEGES
  - DBMS에 적용하라(이 명령어를 실행 안하면 위에 GRANT ... 내용이 적용 안됨

### 4. 생성한 유저로 Database 접속

- mysql -h**HostName** -u**DBusername** -p **DBname**
  - 위의 명령어로 DB에 접속
  - ex) mysql -h127.0.0.1 -uconnectuser -p connectdb
    - Host명은 MySQL이 현재 컴퓨터에 설치된 경우라서 127.0.0.1로 사용(localhost도 가능)
    - db이름이 connectdb
    - db계정이 connectuser
- mysql> quit;
  - DB연결 끊기

### 5. MySQL 버전과 현재 날짜 구하기

- mysql> SELECT version(), CURRENT_DATE;
- 결과:

```
+-----------+--------------+
| version() | current_date |
+-----------+--------------+
| 8.0.15    | 2019-02-16   |
+-----------+--------------+
1 row in set (0.00 sec)
```



### 6. 기타 명령어

- 현재 서버에 존재하는 DB 찾아보기
  - mysql> show databases;
- 사용중인 Database 전환
  - mysql> use **DBname**;
  - 전환 하려면 해당 Database가 존재해야하고, 접속중인 계정이 해당 DB에 권한이 있어야한다.

## 7. Table

- 특징
  - RDBMS의 기본적인 저장 구조. 1개 이상의 Column과 0개 이상의 Row로 구성
  - Column: 단일 종류의 데이터를 나타냄 특정 데이터 타입 및 크기를 가지고 있음
  - Row: Column들의 값의 조합. Record라고 불림. (PK)Primary Key에 의해 구분. PK는 중복이 있으면 안됨
  - Field: Row와 Column의 교차점으로 Field는 데이터를 포함할 수 있고 없을 때는 NULL을 가지고 있음
- 테이블 목록 확인하기
  - show tables;
  - use DBname 명령어를 통해서 DB를 선택하고 위 명령어 입력 시 해당 DB의 모든 table을 보여줌
- 테이블 내의 구조 확인
  - desc **Tablename**;
- SQL 연습을 위한 table 생성과 값의 저장
  - edwith에서 examples**.sql 다운 (examples.sql에는 연습을 위한 table 생성문과 해다 table에 값을 저장하는 입력문이 존재)
  - 다운 받은 파일이 있는 디렉토리로 이동 후
  - mysql -u**Username** -p **DBname** < **examples**.sql


원본 강의: https://www.edwith.org/boostcourse-web
