---
title: "Spring JDBC"
date: 2019-04-22T12:43:40+09:00
archives: "2019"
tags: ["spring", "edwith", "webprogramming"]
author: Joohan Lee

---

## Spring JDBC

### 1. 개념

- JDBC를 이용한 개발에서 반복적으로 등장하는 저수준 세부사항을 처리해준다
- 개발자는 필요한 부분만 개발

| Action                                                      | Spring | Developer |
| ----------------------------------------------------------- | ------ | --------- |
| Connection parameter 정의                                   |        | O         |
| Connection 오픈                                             | O      |           |
| SQL query                                                   |        | O         |
| Parameter 선언과 Parameter Value 제공                       |        | O         |
| Statement 준비와 실행                                       | O      |           |
| (존재한다면) 결과를 반복하는 Loop 설정                      | O      |           |
| 각 Iteration에 대한 작업 수행                               |        | O         |
| Exception Handling - Spring에서 제공하는 Exception으로 변환 | O      |           |
| Connection, Statement, ResultSet 닫기                       | O      |           |



### 2. Spring JDBC 패키지

- org.springframework.jdbc.core
  - JDBC template class와 JDBC Template의 다양한 Callback interface를 포함
- org.springframework.jdbc.datasource
  - Data source의 접근은 쉽게하는 유틸리티 클래스와 JavaEE 컨테이너 외부에 수정되지 않고 운영되는 JDBC 코드와 Test에서 사용할 수 있는 간단한 DataSource 구현체를 포함
- org.springframework.jdbc.object
  - RDBMS 조회, 갱신, 저장등을 Thread-safe하고 재사용 가능한 객체 제공
- org.springframework.jdbc.support
  - SQL Exception변환 기능과 유틸리티 제공

### 3. JDBC Template

- org.springframework.jdbc.core에서 가장 중요한 클래스입니다.
- 리소스 생성, 해지를 처리해서 연결을 닫는 것을 잊어 발생하는 문제 등을 피할 수 있도록 합니다.
- 스테이먼트(Statement)의 생성과 실행을 처리합니다.
- SQL 조회, 업데이트, 저장 프로시저 호출, ResultSet 반복호출 등을 실행합니다.
- JDBC 예외가 발생할 경우 org.springframework.dao패키지에 정의되어 있는 일반적인 예외로 변환시킵니다.



### 4. JDBC Template 예제

- 열의 수 구하기
  - `int rowCount = this.jdbcTemplate.queryForInt("SELECT COUNT(*) FROM t_actor");`
- 변수 바인딩 사용하기
  - `int countOfActorsNamedJoe = this.jdbcTemplate.queryForInt("SELECT COUNT(*) FROM t_actor WHERE first_name = ?", "JOE");`
- String 값으로 결과 받기
  - `String lastName = this.jdbcTemplate.queryForObject("SELECT last_name FROM t_actor WHERE id = ?", new Object[] {1212L}, String.class);
- 한 건 조회하기

```java
Actor actor = this.jdbcTemplate.queryForObject(
    "SELECT first_name, last_name FROM t_actor WHERE id = ?",
    new Object[] {1212L},
    new RowMapper<Actor>() {
        public Actor mapRow(ResultSet rs, int rowNum) throws SQLException {
            Actor actor = new Actor();
            actor.setFirstName(rs.getString("first_name"));
            actor.setLastName(rs.getString("last_name"));

            return actor;
        }
    });
```

- 여러건 조회하기

```java
Actor actor = this.jdbcTemplate.query(
    "SELECT first_name, last_name FROM t_actor",
    new Object[] {1212L},
    new RowMapper<Actor>() {
        public Actor mapRow(ResultSet rs, int rowNum) throws SQLException {
            Actor actor = new Actor();
            actor.setFirstName(rs.getString("first_name"));
            actor.setLastName(rs.getString("last_name"));

            return actor;
        }
    });
```

- 중복 코드 제거 (1건 구하기와 여러건 구하기가 같은 코드에 있을 경우)

```java
public List<Actor> findAllActors() {
    return this.jdbcTemplate.query("SELECT first_name, last_name FROM t_actor", new ActorMapper());
}

private static final class ActorMapper implements RowMapper<Actor> {
    public Actor mapRow(ResultSet rs, int rowNum) throws SQLException {
            Actor actor = new Actor();
            actor.setFirstName(rs.getString("first_name"));
            actor.setLastName(rs.getString("last_name"));

            return actor;
        }
}
```

- INSERT 하기
  - `this.jdbcTemplate.update("INSERT INTO t_actor (first_name, last_name) values (?, ?)", "Leonor", "Watling");`
- UPDATE 하기
  - `this.jdbcTemplate.update("UPDATE t_actor set = ? WHERE id=?"), "Benjo", 5276L);`
- DELETE 하기
  - `this.jdbcTemplate.update("DELETE FROM actor WHERE id=?"), Long.valueOf(actorId));`

### 5. jdbcTemplate 외의 접근방법

- NamedParameterJdbcTemplate - ? 대신 변수명으로 SQL문에 주입할 수 있도록 해줌
- SimpleJdbcTemplate - jdbcTemplae과 NamedParameterJdbcTemplate에서 가장 빈번하게 사용되는 작업을 합쳐놓은 객체
- SimpleJdbcInsert - 쉽게 Insert할 수 있음.
