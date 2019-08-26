# JPA Annotation 정리

## 1. Class에 사용

- @Entity

  - 속성
    - name - Entity명 설정. 따로 설정하지 않으면 class명을 Entity 명으로 함
  - 해당 class를 JPA가 관리하도록 함
  - 주의 사항
    - 기본 생성자는 필수 (parameter가 없는 public 또는 protected constructor)
    - final, enum, interface, inner 클래스에는 사용 불가능
    - 저장할 field를 final처리 불가

- @Table

  - 속성
    - name - 맵핑할 table명
    - catalog - catalog 기능이 있는 DB에서 catalog를 매핑
    - schema - schema 기능이 있는 DB에서 schema를 매핑
    - uniqueConstraints - schema 자동 생성 기능을 사용해서 DDL 생성시에 유니크 제약 조건을 만듬
  - Entity와 맵핑할 Table을 지정
  - 생략하면 Entity명을 Table명으로 사용

- @DynamicUpdate

- - UPDATE문을 보낼 때 모든 필드 업데이트 쿼리를 보내지 않고 변경되는 필드만 보내도록 수정
  - 해당 Annotation을 사용하지 않을 경우 DB와 EntityManager에서 이미 생성된 Query를 재사용할 수 있기 때문에 Column이 30개 이상일 경우 이 Annotation을 붙이는게 효율적이라고 한다

ex) 기본 Update 방식

```java
@Entity
class Person {
	private long id;
  private String name;
  private int age;
  private String address;
}

...
transaction.begin(); // Transaction begin
person.setAge(10);
person.setName("person1");

transaction.commit(); // commit

Result Query
-------------------------------------------
UPDATE PERSON
SET 
	NAME=?
  AGE=?
  ADDRESS=?
WHERE
	id=?
```

Ex) @DynamicUpdate 사용 (Address field가 업데이트 되지 않음)

```java
@DynamicUpdate
@Entity
class Person {
	private long id;
  private String name;
  private int age;
  private String address;
}

...
transaction.begin(); // Transaction begin
person.setAge(10);
person.setName("person1");

transaction.commit(); // commit

Result
-------------------------------------------
UPDATE PERSON
SET 
	NAME=?
  AGE=?
WHERE
	id=?
```

- @DynamicInsert
  - INSERT Query를 보낼 때 Entity의 필드 값이 비어있을 경우 생략

````java
@DynamicInsert
@Entity
class Person {
	private long id;
  private String name;
  private int age;
  private String address;
}

...
transaction.begin(); // Transaction begin
Person person = new Person();
person.setAge(10);
person.setName("person1");

transaction.commit(); // commit

Result
-------------------------------------------
INSERT INTO PERSON (NAME, AGE)
VALUES ('person1', 10)
````

## 2. Field에 사용

- @Id
- @Column
- @ManyToOne
- @JoinColumn
- @Enumerated
  - ENUM 타입을 사용할 때 추가하는 annotation
  - 기본값이 순서대로 받는 숫자 (0부터 … )이기 때문에 (EnumType.STRING)을 추가하길 권장
- @Temporal(TemporalType.TIMESTAMP
  - 날짜 타입을 쓸 때 사용됨
- @Lob
  - byte[]와 char[]의 경우에 사용
- @Transient
  - 해당 field를 맵핑하지 않음

```java
@Entity
@Table(name="PERSON")
public class Person {
	@Id
  @Column(name="ID")
  private long id;
  
  @Column(name="name")
  private String name;
  
  private int age;
  
  @Enumerated(EnumType.STRING)
  private Role roleType;
  
  @Temporal(TemporalType.TIMESTAMP)
  private Date createdDate;
  
  @Temporal(TemporalType.TIMESTAMP)
  private Date lastModifiedDate;
  
  @Lob
  private String description;
}

public enum Role {
  ADMIN, USER
}
```





