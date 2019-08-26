---
title: "마이크로서비스 vs. 모노리스"
date: 2019-02-24T10:39:07+09:00
archives: "2019"
tags: ["K-MOOC", "MicroService"]
author: Joohan Lee
---

## 마이크로서비스 vs. 모노리스

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