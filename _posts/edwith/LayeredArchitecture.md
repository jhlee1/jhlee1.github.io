---
title: "LayeredArchitecture"
date: 2019-05-05T23:10:36+09:00
archives: "2019"
tags: ["spring", "edwith", "webprogramming"]
author: Joohan Lee
---

## Layered Architecture

### 1. Controller에서 중복되는 부분을 처리하려면?

- 별도의 객체로 분리
- 별도의 Method로 분리
- 쇼핑몰에서 게시판에서도 회원정보를 보여주고, 상품 목록 보기에서도 회원정보를 보여주는 경우 회원 정보를 읽어오는 코드는 어떻게 해야 할까?

### 2. Controller와 Service

- 비지니스 로직을 별도의 Service 객체를 구현하도록 하고 Controller는 Service 객체를 사용
- Service 객체란
  - Business Logic을 수행하는 Method를 가지고 있는 객체
  - 하나의 Business Logic은 하나의 Transaction으로 동작

![LayeredArchitecture_Controller_Service](..\..\..\resources\images\edwith\LayeredArchitecture\LayeredArchitecture_Controller_Service.png)

### 3. Transaction

- 하나의 논리적인 작업을 의미

- 아래의 특징을 가짐

- 원자성 (Atomicity)

  - 전체가 성공하거나 실패하는 것을 의미
  - ex) 출금이라는 Business Logic의 경우 아래의 과정을 거침
    - 잔액 조회
    - 출금 금액이 잔액보다 작은지 검사
    - 출금 금액이 잔액보다 작다면 (잔액 - 출금액)으로 수정
    - 언제, 어디서 출금했는지 정보를 기록
    - 사용자에게 출금
  - 위의 순서중 중간(ex. 3번째, 4번째)에서 오류가 발생한경우 앞의 작업을 모두 취소시켜야함. 이를 Rollback이라고 한다.
  - 모든 단계를 성공한 경우에만 위의 작업들이 반영되어야함. 이를 Commit이라고 함
  - Rollback하거나 Commit한 경우 하나의 Transaction이 처리됨

- 일관성 (Consistency)

  - 트랜잭션의 작업 처리 결과가 항상 일관성이 있어야 한다.
  - 트랜잭션이 진행되는 동안에 데이터가 변경되더라도 업데이트된 데이터로 트랜잭션이 진행되는 것이 아니라, 처음에 트랜잭션을 진행하기 위해 참조한 데이터로 진행 -> 각 사용자는 일관성 있는 데이터를 볼 수 있다

- 독립성 (Isolation)
  - 둘 이상의 트랜잭션이 동시에 병행 실행되고 있을 경우에 어느 하나의 트랜잭션이라도 다른 트랜잭션의 연산을 끼어들 수 없음
  - 하나의 특정 트랜잭션이 완료될 때까지, 다른 트랜잭션이 특정 트랜잭션의 결과를 참조할 수 없다

- 지속성 (Durability)
  - Transaction이 성공적으로 완료됬을 경우, 결과는 영구적으로 반영됨
- JDBC Programming에서 트랜잭션 처리 방법
  - DB에 연결된 후 Connection객체의 setAutoCommit메소드에 false
  -  CRUD 실행 후 모두 성공했을 경우 Connection이 가지고 있는 commit()메소드를 호출.

- @EnableTransactionManagement
  - Spring Java Config파일에서 Transaction을 활성화 할 때 사용하는 Annotation
  - Java Config를 사용하게 되면 PlatformTransactionManager 구현체를 모두 찾아서 그 중에 하나를 Mapping해서 사용
  - 특정 Transaction Manager를 사용하고자 한다면 TransactionManagementConfigurer를 Java Config파일에서 구현하고 원하는 Transaction Manager를 return 처리하거나 - 특정 Transaction Manager 객체 생성시 @Primary Annotation을 추가



### 4. 서비스 객체에서 중복으로 호출되는 코드의 처리
- 데이터 엑세스 메소드를 별도의 Repository(Dao) 객체에서 구현하도록 하고 Service는 Repository객체를 사용

![LayeredArchitecture_Layers](..\..\..\resources\images\edwith\LayeredArchitecture\LayeredArchitecture_Layers.png)

- 굳이 나누는 이유: Web 프로젝트를 개발하다가 Window Application을 개발할 경우 Service와 Repository는 그대로 두고 Presentation Layer만 바꾸면 쉽게 생성할 수 있다 -> 설정도 분리하는 것이 좋음

- 설정의 분리
  - Spring 설정 파일을 Presentation Layer쪽과 나머지를 분리
  - web.xml 파일에서 Presentation Layer에 대한 스프링 설정은 DispatcherServlet이 읽도록 하고, 그 외의 설정은 ContextLoaderListener를 통해서 읽어온다.
  - DispatcherServlet을 경우에 따라서 2개 이상 설정할 수 있는데 이 경우에는 각각의 DispatcherServlet의 ApplicationContext가 각각 독립적이기 때문에 각각의 설정 파일에서 생성한 빈을 서로 사용할 수 없다
  - 위의 경우와 같이 동시에 필요한 빈은 ContextLoaderListener를 사용함으로써 공통으로 사용하게 할 수 있다
  - ContextLoaderListener와 DispatcherServlet은 각각 ApplicationContext를 생성하는데, ContextLoaderListener가 생성하는 ApplicationContext가 root context가 되고 DispatcherServlet이 생성한 인스턴스는 root context를 부모로 하는 자식 context가 생성됨
  - 자식 context들은 root context의 설정 bean을 사용할 수 있다
