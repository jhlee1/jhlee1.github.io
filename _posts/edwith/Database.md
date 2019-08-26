---
title: "Database"
date: 2019-02-15T01:31:19+09:00
archives: "2019"
tags: ["edwith", "database", "webProgramming"]
author: Joohan Lee
---

## Database와 DBMS(DataBase Management System)

### 1. 기본 개념

- 데이터의 집합(a Set of Data)
- 여러 응용 시스템(프로그램)들의 통합된 정보들을 저장하여 운영할 수 있는 공용(share) data의 집합
- 효율적으로 저장, 검색, 갱신할 수 있도록 데이터 집합들끼리 연관시키고 조직화되어야 한다.

### 2. 특성

- 실시간 접근성(Real-time Accessability)
  - 사용자의 요구를 즉시 처리
- 계속적인 변화(Continuous Evolution)
  - 정확한 값을 유지하려고 삽입, 삭제, 수정 작업 등을 이용해 데이터를 지속적으로 갱신
- 동시 공유성(Concurrent Sharing)
  - 사용자마다 다른 목적으로 사용하므로 동시에 여러 사람이 동일한 데이터에 접근, 이용
- 내용 참조(Content Reference)
  - 저장한 data의 record의 위치나 주소가 아닌 사용자가 요구하는 data의  내용, 즉 data 값에 따라 참조(reference)할 수 있어야 한다.

### 3. DBMS

- Database를 관리하는 소프트웨어
- 여러 응용 소프트웨어(프로그램) 또는 시스템이 동시에 Database에 접근하여 사용할 수 있게 함
- 필수 3기능
  - 정의기능: DB의 논리적, 물리적 구조를 정의
  - 조작기능: Data를 CRUD하는 기능
  - 제어 기능: DB의 내용 정확성, 안전성을 유지하도록 제어
- EX) Oracle, SQL Server, MySQL, DB2 등

### 4. DBMS의 장단점

- 장점
  - 데이터 중복의 최소화
  - 데이터의 일관성 및 무결성 유지
  - 데이터의 보안 보장
- 단점
  - 운영비가 비싸다
  - 백업 및 복구에 대한 관리가 복잡
  - 부분적 데이터베이스 손실이 전체 시스템을 정지

원본 강의: https://www.edwith.org/boostcourse-web

