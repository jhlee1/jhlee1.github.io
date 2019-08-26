---
title: "SQL Data Definition Language"
date: 2019-02-17T00:44:19+09:00
archives: "2019"
tags: ["edwith", "SQL", "webProgramming", "database"]
author: Joohan Lee
---

## DDL(Data Definition Language)

### 1. 정의

- DB의 schema를 생성, 변경 제거

### 2. MySQL Data type

- Check this link (https://dev.mysql.com/doc/refman/8.0/en/data-type-overview.html). I will summarize later

### 3. Create table

- 기본형

  - CREATE TABLE **tableName**(
    **field1** **type** **\[NULL | NOT NULL\]\[DEFAULT\]\[AUTO_INCREMENT\]**
      	**field2** **type** **\[NULL | NOT NULL\]\[DEFAULT\]\[AUTO_INCREMENT\]**
      	**field3** **type** **\[NULL | NOT NULL\]\[DEFAULT\]\[AUTO_INCREMENT\]**
      	............
      	PRIMARY KEY(**fieldName**)

    );

- NULL, NOT NULL로 필드에 빈 값 허용 여부 결정

- DEFAULT - 입력 값이 없는 경우 초기값 결정

- AUTO_INCREMENT - 자동으로 1씩 증가

- ex)

```
CREATE TABLE EMPLOYEE2(
	empno INTEGER NOT NULL PRIMARY KEY,
	name VARCHAR(10),
	job VARCHAR(9),
	boss INTEGER,
	hiredate VARCHAR(12)
	salary DECIMAL(7,2),
	comm DECIMAL(7, 2),
	deptno INTEGER
);
```



### 4. Edit Table

- 필드 추가
  - ALTER TABLE **tableName**
    ADD **field1** **type** **\[NULL | NOT NULL\]\[DEFAULT\]\[AUTO_INCREMENT\]**;

- ex) ADD

```
ALTER table book
	ADD author varchar(20);
```

- 필드 삭제
  - ALTER TABLE **tableName**
    DROP **fieldName**

- ex) DROP

```
ALTER table book
	DROP price
```

- 필드 수정
  - ALTER TABLE **tableName**
    CHANGE **oldFieldName** **newFieldName** **type** **\[NULL | NOT NULL\]\[DEFAULT\]\[AUTO_INCREMENT\]**;
- ex) CHANGE

```
ALTER table EMPLOYEE2
	CHANGE deptno dept_no int(11);
```

- Table 이름 변경
  - ALTER TABLE **tableName** rename **newTableName**
- ex) RENAME

```
ALTER table EMPLOYEE2 rename EMPLOYEE3
```

- Table 삭제
  - DROP TABLE **tableName**
  - 단, 다른 테이블이 해당 테이블을 참조하고 있는 경우 지워지지 않음. 생성한 순서 반대로 삭제해야됨
  - ex) DROP table

```
DROP TABLE EMPLOYEE2
```

원본 강의: https://www.edwith.org/boostcourse-web
