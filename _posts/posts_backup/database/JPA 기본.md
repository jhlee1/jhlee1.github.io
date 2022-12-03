# JPA 기본

## I. JPA 등장 배경

- OOP <-> RDB 간의 패러다임 불일치
- 객체지향적으로 모델링할수록 지저분해지는 코드
- 상속과 객체간의 의존성을 구현하기 어려움
  - 객체가 연결되어있는 경우 해당 객체를 DB에 넣기 위해 INSERT문을 연결된 개수만큼
  - 조회의 경우 연결이 늘어날수록 JOIN의 경우의 수가 기하급수적으로 늘어남
## II.  개념![img](https://lh5.googleusercontent.com/sOCW3lDFEcaDvnILjnhtlnrmsoiznhdFwyfUY30eOaMZy7by0hnZYpJFxjEicm3P3w-nfYh2O0gEZxkMTXS-Cclftlfm0yJR6sggW96z37qs97piKDJW3KLO0GZdqUWCx8FJ2UUP)

- JAVA Application과 JDBC 사이에서 다리 역할
- Model을 만들면 JPA에서 직접 Query를 만들어서 보내기 때문에 관계형 데이터베이스에 쿼리를 직접 보내야 하는 것을 신경쓰지 않아도된다
- 객체지향적 모델링에 집중할 수 있도록 도와줌

    - ex) Member 와 Team이 연결관계에 있는 경우 jpa.persist(member)로 Member객체를 넣어주는 경우 JPA에서 INSERT Member … 와 INSERT Team … 을 보냄
- 유지보수가 편하다
  - 필드 명이 변경된 경우 각 Query의 Parameter를 직접 하나하나 변경하지 않아도됨
    - ex) name을 first_name과 last_name으로 분리하여 저장하게 된 경우
    - SELECT name FROM member -> SELECT first_name, last_name FROM member
    - 위와 같은 경우 name을 사용하고 있던 모든 query를 손수 변경해야함
## III. 성능 최적화
- 1차 캐시와 동일성(Identity) 보장
  - 같은 Transaction안에서는 같은 Entity를 반환
  - DB Isolation Level이 Read Commit이어도 Application에서 Repeatable Read 보장
  - ex) 기존 직접 SQL Query 작성방식
```java
  String memberId = “100”;Member m1 = memberDAO.getMember(memberId); // SQL을 보냄 1
  Member m2 = memberDAO.getMember(memberId); // SQL을 보냄 2

// m1 != m2 매 번 SQL을 보내고 받아온 후 새로 객체를 생성하기 때문에 서로 다른 객체 

String memberId = “100”;
Member m1 = jpa.find(Member.class, memberId); // SQL을 보냄
Member m2 = jpa.find(Member.class, memberId); // cache에서 가져옴

// m1 == m2 사실 SQL을 한번만 보내고 두번째는 cache에서 가져오기 때문에 같은 객체가 유지됨
```

- Transaction을 지원하는 쓰기 지연
	- INSERT
		-	commit할 때까지 INSERT SQL을 모았다가 JDBC BATCH SQL 기능을 사용해서 한번에 SQL 전송
		-	ex)
		```java
		transaction.begin(); // [트랜잭션] 시작 em.persist(memberA);
		em.persist(memberB); em.persist(memberC); //여기까지 INSERT SQL을 데이터베이스에 보내지 않는다.
		
		//커밋하는 순간 데이터베이스에 INSERT SQL을 모아서 보낸다. 
		transaction.commit(); // [트랜잭션] 커밋
		```
    - UPDATE
	    - UPDATE, DELETE로 인한 로우(ROW)락 시간 최소화
    	- 트랜잭션 커밋 시 UPDATE, DELETE SQL 실행하고, 바로 커밋 
    	- ex)
    	```java
      transaction.begin(); // [트랜잭션] 시작 
      changeMember(memberA); 
      deleteMember(memberB); 
      
      // 비즈니스_로직_수행(); 비즈니스 로직 수행 동안 DB 로우 락이 걸리지 않는다. 

			//커밋하는 순간 데이터베이스에 UPDATE, DELETE SQL을 보낸다. 
			transaction.commit(); // [트랜잭션] 커밋
			```
	- 지연 로딩과 즉시 로딩
		- 일단 지연 로딩으로 작업 후 Query를 부르는 횟수가 너무 많다 싶으면 즉시 로딩을 사용하는 것도 괜찮은 방법
		- 지연 로딩: 객체가 실제 사용될 때 로딩
		- ex) 
		```java
		Member member = memberDAO.find(memberId); // SELECT * FROM MEMBER
		Team team = member.getTeam();
		String teamName = team.getName(); // SELECT * FROM TEAM
		```
		- 즉시 로딩: JOIN SQL로 한번에 연관된 객체까지 미리 조회
		- ex)
		```java
		Member member = memberDAO.find(memberId); // SELECT M.*, T.* FROM MEMBER JOIN TEAM ...
		Team team = member.getTeam();
		String teamName = team.getName();
		```
		
## VI. JPA 개발 순서
1. Entity Manager Factory 설정
2. Entity Manager 설정
3. Transaction
4. Business Logic (CRUD)
## V. Entity Manager
- EntityManagerFactory는 하나만 생성해서 어플리케이션 전체에서 공유
- Entity Manager는 Thread간에 공유하면 안됨 (사용하고 버리기)
- JPA의 모든 데이터 변경은 Transaction안에서 실행

참고 링크 
https://www.youtube.com/watch?v=WfrSN9Z7MiAhttps://jogeum.net/7?category=766565