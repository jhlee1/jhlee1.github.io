---
title: "SQL(Data Manipulation Language)"
date: 2019-02-17T00:35:56+09:00
archives: "2019"
tags: ["edwith", "SQL", "webProgramming", "database"]
author: John SMITH
---

## DML(Data Manipulation Language)

### 1. SELECT

- 기본형
  - SELECT(DISTINCT) **column** (ALIAS)
    FROM **table**;
- SELECT - 찾는 데이터(Column)을 나열
- DISTINCT - 중복행 제거
- ALIAS - 나타날 Column에 대한 다른 이름 부여
- FROM - 선택한 Column이 있는 테이블을 명시

- ex) 전체 데이터 검색
  - SELECT * FROM DEPARTMANT;

```
+--------+------------+----------+
| deptno | name       | location |
+--------+------------+----------+
|     10 | ACCOUNTING | NEW YORK |
|     20 | RESEARCH   | DALLAS   |
|     30 | SALES      | CHICAGO  |
|     40 | OPERATIONS | BOSTON   |
+--------+------------+----------+
4 rows in set (0.00 sec)
```

-  원하는 column 내용 가져오기(어떤 Column이 있는지 desc 명령어로 확인)
  - SELECT deptno 부서번호, name 부서명 from department;
  - SELECT deptno as 부서번호, name as 부서명 from department;

```
+----------+------------+
| 부서번호 | 부서명     |
+----------+------------+
|       10 | ACCOUNTING |
|       20 | RESEARCH   |
|       30 | SALES      |
|       40 | OPERATIONS |
+----------+------------+
4 rows in set (0.00 sec)
```

- CONCAT을 이용한 문자열 결합
  - SELECT CONCAT(empno, '-', deptno) AS '사번-부서번호'  FROM employee;

```
+---------------+
| 사번-부서번호 |
+---------------+
| 7782-10       |
| 7839-10       |
| 7934-10       |
| 7369-20       |
| 7566-20       |
| 7788-20       |
| 7876-20       |
| 7902-20       |
| 7499-30       |
| 7521-30       |
| 7654-30       |
| 7698-30       |
| 7844-30       |
| 7900-30       |
+---------------+
14 rows in set (0.00 sec)
```

- DISTINCT를 이용한 중복 제거
  - SELECT DISTINCT deptno FROM employee;

```
+--------+
| deptno |
+--------+
|     10 |
|     20 |
|     30 |
+--------+
```

- ORDER BY를 이용한 정렬
  - SELECT empno, name FROM employee ORDER BY name; (name column 기준으로 오름차순)
  - SELECT empno, name FROM employee ORDER BY 2 DESC; (2번째 column 기준으로 내림차순)
  - ASC - 오름차순 정렬, 기본값
  - DESC - 내림차순

```
+-------+--------+
| empno | name   |
+-------+--------+
|  7876 | ADAMS  |
|  7499 | ALLEN  |
|  7698 | BLAKE  |
|  7782 | CLARK  |
|  7902 | FORD   |
|  7900 | JAMES  |
|  7566 | JONES  |
|  7839 | KING   |
|  7654 | MARTIN |
|  7934 | MILLER |
|  7788 | SCOTT  |
|  7369 | SMITH  |
|  7844 | TURNER |
|  7521 | WARD   |
+-------+--------+
14 rows in set (0.00 sec)
```

- WHERE를 이용한 특정 Row 검색

  - SELECT(DISTINCT) column(ALIAS)
    FROM tablename
    WHERE condition
    ORDER BY column or expression (ASC or DESC)
  - ex) SELECT * FROM employee WHERE title = 'Staff'
  - BETWEEN
    - SELECT * FROM employee WHERE salary BETWEEN 1000 AND 2000
  - 비교 연산자(<, >, =)
    - SELECT name, hiredate FROM employee where hiredate < '1981-01-01';
  - IN 사용
    - SELECT name, deptno FROM employee WHERE deptno in (10, 30);
    - deptno가 10 또는 30인 employee의 name과 deptno 검색
  - AND
    - SELECT name, deptno FROM employee WHERE deptno in (10, 30) AND hiredate < '1981-01-01';
    - AND로 여러개의 조건을 합칠 수 있음
  - LIKE
    - Wild Card를 사용하여 특정 문자를 포함하는가에 대한 조건
    - %는 0에서부터 여러개의 문자열을 나타냄
    - _는 단 하나의 문자를 나타내는 Wild Card
    - ex1) SELECT name, job FROM employee WHERE name like 'A%';
    - name이 'A'로 시작하는 직원의 name과 job을 검색
    - ex2) SELECT name, job FROM employee WHERE name like '%A';
    - name이 'A'로 끝나는 직원의 name과 job을 검색
    - ex3) SELECT name, job FROM employee WHERE name like '%A%';
    - name 에 'A'를 포함하는 직원의 name과 job을 검색
    - ex3) SELECT name, job FROM employee WHERE name like '_A%';
    - name의 character가 'A'인 직원의 name과 job을 검색
- 함수 사용
  - UPPER, UCASE
    - 대문자화
    - ex) SELECT UPPER('SEoul'), UCASE('seOUL');
    - FROM 다음에 table이 없는 경우 table에서 조회하는 것이 아니다.
  - LOWER, LCASE
    - 소문자화
    - ex) SELECT LOWER(name) from employee;
  - SUBSTRING
    - index의 시작을 0부터가 아니라 1부터 함
    - ex) SELECT SUBSTRING('Happy Day', 3, 2)

  ```
  +------------------------------+
  | SUBSTRING('Happy Day', 3, 2) |
  +------------------------------+
  | pp                           |
  +------------------------------+
  1 row in set (0.00 sec)
  ```

  - LPAD, RPAD
    - 문자열이 지정된 자리만큼 차지하도록 되도록 정해진 문자열로 (왼쪽, 오른쪽)채우기
    - ex) SELECT LPAD('hi', 5, '?'), LPAD('joe', 7, '*');

  ```
  +--------------------+----------------------+
  | LPAD('hi', 5, '?') | LPAD('joe
  ', 7, '*') |
  +--------------------+----------------------+
  | ???hi              | ***joe
                |
  +--------------------+----------------------+
  1 row in set (0.00 sec)
  ```

  - TRIM, LTRIM, RTRIM
    - 공백 없애기
    - SELECT LTRIM('hello'), RTRIM('hello');
  - ABS()
  - MOD(n, m) n % m
  - FLOOR(x)

  - CEILING(x)
  - ROUND(x)
  - POW(x,y)
  - GREATEST(x,y,...)
  - LEAST(x,y,...)
  - CURDATE(),CURRENT_DATE
  - CURTIME(), CURRENT_TIME
  - NOW(), SYSDATE() , CURRENT_TIMESTAMP
  - DATE_FORMAT(date,format) : 입력된 date를 format 형식으로 반환
  - PERIOD_DIFF(p1,p2) : YYMM이나 YYYYMM으로 표기되는 p1과 p2의 차이 개월을 반환

  - CAST를 이용한 type 변환

    - 기본형:
      CAST(expression As type)
      CONVERT(expression, type)
      CONVERT(expr USING transcoding_name)
    - MySQL 타입:
      - BINARY
      - CHAR
      - DATE
      - DATETIME
      - SIGNED {INTEGER}
      - TIME
      - UNSIGNED {INTEGER}
    - ex)
      SELECT CAST(NOW() as DATE);
      SELECT CAST(1-2 as unsigned;

    - COUNT(expr)
    - AVG(expr)
    - MIN(expr)
    - MAX(expr)
    - SUM(expr)
    - GROUP_COUNCAT(expr)
    - VARIANCE(expr)
    - STDDEV(expr)
    - ex1) SELECT AVG(salary), SUM(salary) FROM employee WHERE deptno = 30;
      - deptno가 30인 직원의 salary평균과 합을 검색
    - ex2)  SELECT AVG(salary), SUM(salary) FROM employee GROUP BY deptno;
      - deptno별로 직원의 salary평균과 합을 검색

  - INSERT 문

    - 사용 방법
      - INSERT INTO **tablename**(**field1**, **field2**, **field3**, ...)
        VALUES (**field1**에 넣을 값, **field2**에 넣을 값,  **field3**에 넣을 값, ...)
      - INSERT INTO **tablename**
        VALUES (**field1**에 넣을 값, **field2**에 넣을 값,  **field3**에 넣을 값, ...)
    - Default 값을 설정한 필드의 경우 생략 가능
    - 필드명을 생략한 경우 VALUES( ... )에 Default 값이 설정된 필드를 제외한 모든 필드에 넣을 값을 채워줘야함. 순서는 DESC를 이용해서 참고
    - ex)
      INSERT INTO ROLE(role_id, description) values (200, 'CEO');
      INSERT INTO ROLE values (200, 'CEO');
      - ROLE이라는 table에 role_id는 200, description은 CEO로 row를 생성

  - UPDATE 문

    - 기본형
      - UPDATE **tablename**
        SET **field1=val1, field2=val2, ...**
        WHERE **condition**
    - WHERE을 통해 특정 row를 변경
    - WHERE이 없는 경우 전체 ROW를 변경하게 되므로 주의
    - ex)
      UPDATE role
      SET description ='CTO'
      WHERE role_id = 200;
    - role_id가 200인 role의 description을 'CTO'로 변경

  - DELETE 문

    - 기본형
      - DELETE
        	FROM **tablename**
        WHERE **condition**
    - WHERE이 없는 경우 전체 ROW를 삭제하게 되므로 주의
    - ex) DELETE FROM role WHERE role_id = 200;
      - role_id가 200인 row 삭제

원본 강의: https://www.edwith.org/boostcourse-web
