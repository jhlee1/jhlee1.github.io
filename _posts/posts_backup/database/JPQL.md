# JPA에서 Query 사용하기

## 1. Native Query

- JPA에선 native query를 직접 보낼 수 있음
- JPA를 통해서가 아니라 JDBC와 MyBatis 같은 SQL Mapper 프레임워크를 직접 사용해도 됨

## 2. JPQL

- 객체지향 쿼리언어
-  Entity보다 Object에 초점이 맞춰진 문법
- SQL을 추상화해서 특정 데이터베이스에 종속되지 않음
- Entity 직접 조회, 묵시적 조인, 다형성 지원

```java
EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("example"); // persistence.xml의 persistence-unit중 example을 찾아서 설정
EntityManager entityManager = entityManagerFactory.createEntityManager();
String query = "SELECT ac FROM Account ac WHERE ac.username = 'hello'";
List<Member> resultList = entityManager.createQuery(jpql, Account.class).getResultList();
  
```



## 2. Criteria Query

- JPQL을 편하게 작성하도록 도와주는 API, 빌더 클래스 모음

## 3. Query DSL

- Criteria 처럼 JPQL을 편하게 작성하도록 도와주는 API, 빌더 클래스 모음이지만 비표준 OpenSource Framework
