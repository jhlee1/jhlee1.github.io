---
title: "마이크로서비스 vs. 모노리스"
date: 2019-02-24T10:39:07+09:00
archives: "2019"
tags: ["K-MOOC", "msa"]
categories: [msa]
---

## I. 마이크로서비스 vs. 모노리스

### 1. 개념

- 하나의 어플리케이션을 여러개의 서비스 집합으로 구성하는 것
- Service Oriented Architecture의 연장선

### 2. SOA VS. MSA

|               | MSA                                                          | SOA                                                          |
| ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 공통점        | 서비스 중심 설계 지향                                        |                                                              |
| Ownership     | 하나의 독립된 팀에서 개발과 관리                             | 여러 팀의 협업(서비스 공유를 위한 미들웨어)                  |
| Size          | SOA 대비 작음                                                | MSA대비 큼                                                   |
| Share         | API로 공유                                                   | UDDI로 공유                                                  |
| Communication | Restful API                                                  | WSDL, SOAP 통신                                              |
| Storage       | 서비스 상 별도의 저장소를 두고 인터페이스를 통한 데이터 캡슐화를 지향 -> 쉬운 분리 및 대체 가능 | 서비스상 저장소를 분리하지 않음 -> 통합 저장소 사용 시 서비스 분리가 어렵다. |

### 3. Martin Fowler가 정의한 MSA

- Monolithic 구조
  - 하나의 단위로 개발되는 일체식 어플리케이션 설계
  - User Interface <-> Application Server(단일 실행체, 아무리 작은 변화라도 새로운 버전을 전체 빌드하여 배포함) <-> Database
  - 단일 프로세스에서 실행
  - 확장 필요시 덩어리를 복사하여 수평확장 가능(Application 서버를 통째로 복사)
    - User Interface <-> Application Server1, Application Server2, Application Server3, Application Server4<-> Database
    - 시스템에 작은 변화가 생길 시 모든 모노리스를 다시 빌드, 배포
    - 특정 기능만 확장 불가, 어플리케이션 동시 확장 필수
- Micro Service 구조
  - 서버쪽 여러 개로 구성, 각각의 서비스가 프로세스 위에 별도로 올라감(여러 개의 서비스가 모여 어플리케이션 구성)
  - User Interface <->  Service A, Service B, Service C, Service C,  <-> DB for A, DB for B, DB for C
  - 확장 시, 특정 기능별 독립적으로 확장 가능
  - 특정 서비스의 변경 시, 서비스만 빌드 , 배포
  - 독립적으로 서로 다른 언어로 개발 가능 -> 각 서비스를 서로 다른 팀이 개발 및 관리 가능
- Martin Fowler의 정의
  - 여러 개의 작은 서비스 집합으로 개발하는 접근 방법
  - 각 서비스는 개별 프로세스에서 실행
  - HTTP 자원 API 같은 가벼운 수단을 사용하여 통신
  - 서비스는 Biz 기능 단위로 구성
    - 중앙 집중적 관리 최소화
    - 자동화된 배포 방식을 이용하여 독립적으로 배포
  - 서비스 마다 서로 다른 언어와 데이터 저장 기술을 사용할 수 있음
    - Polyglot - 인터페이스 만족 시 서비스 안의 기술은 자율적으로 선택할 수 있다.
    - ex) 
      상품 검색 서비스는 Node.js + MongoDB로 개발
      주문 서비스는 C# + Maria DB로 개발
      상품 서비스는 Java + Oracle DB로 개발

## II. Biz 민첩성과 아키텍처 요건

### 1. 11.6초

- Amazon이 Application을 배포하는 주기
- 일반 온라인 기업
  서비스 기획 -> 서비스 지원용 어플리케이션 개발 -> (개선 사항 발생) -> 수정 후 배포
- vs. 국내 쇼핑몰 배포 주기 - 3일
- 최근 빠른 비지니스의 변화로 빠르게 대응할 능력이 필요함

### 2. 개발 문화의 변화 (DevOps)

- "The traditional model is that you take your software to the wall that separates development and operations, and throw it over and then forget about it. Not at Amazon. You build it, you run it." - Werner Vogels (아마존 CTO)
- 기존 - 개발조직과 운영 조직 분리 -> 최근 - 개발과 운영을 함께 진행
- DevOps 조직의 특징
  - 개발 시 빠른속도
  - 전체적 운영의 책임감
  - 문제 발생시 쉬운 개선

### 3. 개발 환경이 이루는 물리적인 하드웨어 인프라

- 기존 - 인프라 구축을 위한 많은 시간이 필요함 (N주 ~ N달)
  - 컴퓨터 장비 설치
  - 서버 장비 구입
  - 네트워크 연결 및 운영체제 설치
- 최근 - 클라우드 환경의 등장

### 4. 클라우드 인프라

- 필요에 따라 쉽고 편한 서비스 형태로 사용
  - 사용자 측면 - 사용한 만큼 과금(Application)
  - 개발자 측면 - 필요한 만큼 과금(Application이 올라가는 Infra)
- ex) AWS, Google Cloud, Azure, Bluemix, 클라우드Z

### 5. 어플리케이션 측면에서의 필요성 (ex 타임 세일)

- Problem: 이벤트 참가용 사용량 증가 -> 전체 서비스 성능 저하
- 해결 방법
  - Scale-up: 사용량 증가에 따른 물리적 인프라 사전 준비 또는 높은 용량의 장비로 대체
  - Scale-out: 전체 시스템을 복사하여 증설 -> 클라우드에서 많이 사용하는 방식
- 마이크로 서비스의 필요성
  - 일부 서비스만 확장이 필요한데 전체 서비스를 복사하는 것은 자원의 낭비
  - 각 서비스를 독립적으로 분리하여 필요한 부분만 확장 ex) 타임 서비스

