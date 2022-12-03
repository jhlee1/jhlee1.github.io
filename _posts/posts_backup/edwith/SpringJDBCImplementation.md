---
title: "Spring JDBC Implementation"
date: 2019-04-25T12:43:40+09:00
archives: "2019"
tags: ["spring", "edwith", "webprogramming"]
author: Joohan Lee
---

## Spring JDBC 구현하기

### 1. DTO (Data Transfer Object)

- 계층간의 데이터 교환을 위한 Bean이다
- 계층 - Controller View, Business Layer, Persistence Layer를 의미
- 일반적으로 DTO는 로직을 갖고 있지 않고 순수한 Data 객체

### 2. DTO의 예

- field, getter, setter를 가진다.
- 추가적으로, toString(), equals(), hashCode() 등의 Object가 가진 method를 @Override할 수 있다.

```java
public class ActorDTO {
    private Long id;
    private String firstName;
    private String lastName;

    public String getFirstName() {
        return this.firstName;
    }

    ...

}
```

### 3. DAO (Data Access Object)

- 데이터를 조회하거나 조작하는 기능을 전담하도록 만든 객체
- Database를 조작하는 기능을 전담하는 목적으로 만들어짐

### 4. Connection Pool이란?

- DB연결은 비용이 많이 든다.
- Connection Pool은 미리 Connection을 여러개 맺어둔다
- Connection이 필요할 때 Client는 Connection Pool에게 빌려서 사용한 후 반납
- 반납하지 않을 경우 계속 연결이 유지되서 다른 Client가 Connection을 얻어올 수 없음



![ConnectionPool](..\..\..\resources\images\edwith\SpringJDBCImplementation\ConnectionPool.jpg)


### 5. Data Source

- Connection Pool은 여러개가 생성될 수 있는데 그러한 Connection Pool을 관리하는 목적으로 사용되는 객체
- Connection을 얻어오고 반납하는 등의 작업을 수행



## 구현해보기

- 구현할 구조

![Spring_JDBC__DAO](..\..\..\resources\images\edwith\SpringJDBCImplementation\Spring_JDBC__DAO.png)



### 6. Database 연결 가져오기

- AnnotationConfig을 이용해서 DataSource 객체 생성 후 연결 가져오기
- 우선 새 프로젝트를 만들고 pom.xml에 plugin과 dependency추가

pom.xml

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>lee.joohan</groupId>
	<artifactId>daoexam</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>daoexam</name>
	<url>http://maven.apache.org</url>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<spring.version>4.3.5.RELEASE</spring.version>
	</properties>

	<dependencies>
		<!-- Spring -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-context</artifactId>
			<version>${spring.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-jdbc</artifactId>
			<version>${spring.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-tx</artifactId>
			<version>${spring.version}</version>
		</dependency>

		<!-- basic data source -->
		<dependency>
			<groupId>org.apache.commons</groupId>
			<artifactId>commons-dbcp2</artifactId>
			<version>2.1.1</version>
		</dependency>


		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>5.1.45</version>
		</dependency>

		<dependency>
			<groupId>org.apache.maven.plugins</groupId>
			<artifactId>maven-resources-plugin</artifactId>
			<version>2.4.3</version>
		</dependency>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>3.8.1</version>
			<scope>test</scope>
		</dependency>
	</dependencies>
	<build>
		<pluginManagement>
			<plugins>
				<plugin>
					<groupId>org.apache.maven.plugins</groupId>
					<artifactId>maven-compiler-plugin</artifactId>
					<version>3.6.1</version>
					<configuration>
						<source>1.8</source>
						<target>1.8</target>
					</configuration>
				</plugin>
			</plugins>
		</pluginManagement>
	</build>
</project>

```

- 전체 App의 Configuration을 담당하는 ApplicationConfig 생성

ApplicationCofig.java

```java
package lee.joohan.daoexam.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;

@Configuration
@Import(DBConfig.class) // 모든 설정을 ApplicatioConfig에 넣을 수 없기 때문에 DBConfig에 DB관련 설정은 따로 빼줌
public class ApplicationConfig {

}

```

- DataSource 객체 설정을 위한 DBConfig 생성

DBConfig.java

```java
package lee.joohan.daoexam.config;

import javax.sql.DataSource;

import org.apache.commons.dbcp2.BasicDataSource;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.transaction.annotation.EnableTransactionManagement;

@Configuration
@EnableTransactionManagement
public class DBConfig {
	private String driverClassName = "com.mysql.jdbc.Driver";
	private String url = "jdbc:mysql://localhost:3306/connectdb?useUnicode=true&character";
	private String username = "connectuser";
	private String password = "password123";

	@Bean
	public DataSource dataSource() {
		BasicDataSource dataSource = new BasicDataSource();

		dataSource.setDriverClassName(driverClassName);
		dataSource.setUrl(url);
		dataSource.setUsername(username);
		dataSource.setPassword(password);

		return dataSource;
	}
}

```

- 연결이 잘 되었는지 검사하기 위해 Test 클래스 생성

DataSourceTest.java

```java
package lee.joohan.daoexam;

import java.sql.Connection;

import javax.sql.DataSource;

import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

import lee.joohan.daoexam.config.ApplicationConfig;

public class DataSourceTest {

	//DB Config에서 설정한 DataSource 객체를 가져와서 연결 부르기
	public static void main(String[] args) {
		ApplicationContext applicationContext = new AnnotationConfigApplicationContext(ApplicationConfig.class);
		DataSource dataSource = applicationContext.getBean(DataSource.class);
		Connection connection = null;

		try {
			connection = dataSource.getConnection();
			if (connection != null) {
				System.out.println("Connection Success");
			}
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			if (connection != null) {
				try {
					connection.close();
				} catch (Exception e) {
					e.printStackTrace();
				}
			}
		}
	}
}
```

### 7. SelectAll 구현

- Role DTO 생성하기

Role.java

```java
package lee.joohan.daoexam.dto;

public class Role {
	private int roleId;
	private String description;

	 public int getRoleId() {
		return roleId;
	}
	public void setRoleId(int roleId) {
		this.roleId = roleId;
	}
	public String getDescription() {
		return description;
	}
	public void setDescription(String description) {
		this.description = description;
	}

	@Override
	public String toString() {
		return "Role [roleId=" + roleId + ", description=" + description + "]";
	}
}

```

- Role DAO 생성하기

RoleDao.java

```java
package lee.joohan.daoexam.dao;

import java.util.Collections;
import java.util.List;

import javax.sql.DataSource;

import org.springframework.jdbc.core.BeanPropertyRowMapper;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate;
import org.springframework.stereotype.Repository;

import lee.joohan.daoexam.dto.Role;

@Repository
public class RoleDao {
	private NamedParameterJdbcTemplate namedParameterJdbcTemplate;
	private RowMapper<Role> rowMapper = BeanPropertyRowMapper.newInstance(Role.class); 	//Java의  변수 표기방식과 DBMS의 Column name 표기 방식을 서로 호환해줌 ex) Java- roleId, DBMS - role_id

	public RoleDao(DataSource dataSource) {
		this.namedParameterJdbcTemplate = new NamedParameterJdbcTemplate(dataSource);
	}

	public List<Role> selectAll() {
		return namedParameterJdbcTemplate.query(RoleDaoSqls.SELECT_ALL, Collections.emptyMap(), rowMapper); // 2nd Parameter: SQL에서 parameter에 바인딩 할 값들을 넘겨주는 목적
	}


}
```

- SQL문을 저장할 Class 생성

RoleDaoSqls.java

```
package lee.joohan.daoexam.dao;

public class RoleDaoSqls {
	public static final String SELECT_ALL = "SELECT role_id, description FROM role order by role_id";
}

```

- ApplicationConfig에 `@ComponentScan(basePackages = {"kr.or.connect.daoexam.dao"}) ` 어노테이션을 추가해서 DAO Bean 만들어주기

ApplicationConfig.java

```java
package lee.joohan.daoexam.config;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;

@Configuration
@ComponentScan(basePackages = {"kr.or.connect.daoexam.dao"}) //base packages에 있는 Bean들을 읽어와서 객체를 생성해줌
@Import(DBConfig.class)
public class ApplicationConfig {

}

```

- Select All 테스트 하기

SelectAllTest.java

```java
package lee.joohan.daoexam;

import java.util.List;

import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

import lee.joohan.daoexam.config.ApplicationConfig;
import lee.joohan.daoexam.dao.RoleDao;
import lee.joohan.daoexam.dto.Role;

public class SelectAllTest {

	//DB Config에서 설정한 DataSource 객체를 가져와서 연결 부르기
	public static void main(String[] args) {
		ApplicationContext applicationContext = new AnnotationConfigApplicationContext(ApplicationConfig.class);
		RoleDao roleDao = applicationContext.getBean(RoleDao.class);

		List<Role> list = roleDao.selectAll();

		for (Role role: list) {
			System.out.println(role);
		}
	}
}
```

### 8. Insert 추가하기

- DAO에 `SimpleJdbcInsert` 와 `Insert method`만 추가하면 됨

RoleDao.java

```java
package lee.joohan.daoexam.dao;

import java.util.Collections;
import java.util.List;

import javax.sql.DataSource;

import org.springframework.jdbc.core.BeanPropertyRowMapper;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate;
import org.springframework.stereotype.Repository;

import lee.joohan.daoexam.dto.Role;

@Repository
public class RoleDao {
	private NamedParameterJdbcTemplate namedParameterJdbcTemplate;
    private SimpleJdbcInsert insertAction;
	private RowMapper<Role> rowMapper = BeanPropertyRowMapper.newInstance(Role.class); 	//Java의  변수 표기방식과 DBMS의 Column name 표기 방식을 서로 호환해줌 ex) Java- roleId, DBMS - role_id

	public RoleDao(DataSource dataSource) {
		this.namedParameterJdbcTemplate = new NamedParameterJdbcTemplate(dataSource);
        this.insertAction = new SimpleJdbcInsert(dataSource)
            .withTableName("role");
	}

	public List<Role> selectAll() {
		return namedParameterJdbcTemplate.query(RoleDaoSqls.SELECT_ALL, Collections.emptyMap(), rowMapper); // 2nd Parameter: SQL에서 parameter에 바인딩 할 값들을 넘겨주는 목적
	}

    public int insert(Role role) {
        // 원래는 primary key를 자동으로 생성하도록 구현하지만 일단은 단순하게 primary key 직접 넣어주도록하자
        SqlParameterSource params = new BeanPropertySqlParameterSource(role);

        return insertAction.execute(params);
    }
}
```

- Insert 테스트 해보기 - Test 클래스 생성

JDBCTest.java

```java
package lee.joohan.daoexam;

import java.util.List;

import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

import lee.joohan.daoexam.config.ApplicationConfig;
import lee.joohan.daoexam.dao.RoleDao;
import lee.joohan.daoexam.dto.Role;

public class JDBCTest {

	//DB Config에서 설정한 DataSource 객체를 가져와서 연결 부르기
	public static void main(String[] args) {
		ApplicationContext applicationContext = new AnnotationConfigApplicationContext(ApplicationConfig.class);
		RoleDao roleDao = applicationContext.getBean(RoleDao.class);
        Role role = new Role();

        role.setRoleId(500);
        role.setDescription("CEO");

		int count = roleDao.insert(role);

        System.out.println("Inserted: " + count);
	}
}
```

### 9. Update 추가하기

- SQL Query 추가하기

RoleDaoSqls.java

```java
package lee.joohan.daoexam.dao;

public class RoleDaoSqls {
	public static final String SELECT_ALL = "SELECT role_id, description FROM role order by role_id";
    public static final String UPDATE = "UPDATE role set description = :description where role_id = :roleId";
}
```

- DAO에 Update 추가

RoleDao.java

```java
package lee.joohan.daoexam.dao;

import java.util.Collections;
import java.util.List;

import javax.sql.DataSource;

import org.springframework.jdbc.core.BeanPropertyRowMapper;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate;
import org.springframework.stereotype.Repository;

import lee.joohan.daoexam.dto.Role;

@Repository
public class RoleDao {
	private NamedParameterJdbcTemplate namedParameterJdbcTemplate;
    private SimpleJdbcInsert insertAction;
	private RowMapper<Role> rowMapper = BeanPropertyRowMapper.newInstance(Role.class); 	//Java의  변수 표기방식과 DBMS의 Column name 표기 방식을 서로 호환해줌 ex) Java- roleId, DBMS - role_id

	public RoleDao(DataSource dataSource) {
		this.namedParameterJdbcTemplate = new NamedParameterJdbcTemplate(dataSource);
        this.insertAction = new SimpleJdbcInsert(dataSource)
            .withTableName("role");
	}

	public List<Role> selectAll() {
		return namedParameterJdbcTemplate.query(RoleDaoSqls.SELECT_ALL, Collections.emptyMap(), rowMapper); // 2nd Parameter: SQL에서 parameter에 바인딩 할 값들을 넘겨주는 목적
	}

    public int insert(Role role) {
        // 원래는 primary key를 자동으로 생성하도록 구현하지만 일단은 단순하게 primary key 직접 넣어주도록하자
        SqlParameterSource params = new BeanPropertySqlParameterSource(role);

        return insertAction.execute(params);
    }

    public int update(Role role) {
        SqlParameterSource params = new BeanPropertySqlParameterSource(role);

        return jdbc.update(RoleDaoSqls.UPDATE, params)
    }
}
```

- Test 해보기 - JDBCTest에 update 테스트 추가하기

JDBCTest.java

```java
package lee.joohan.daoexam;

import java.util.List;

import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

import lee.joohan.daoexam.config.ApplicationConfig;
import lee.joohan.daoexam.dao.RoleDao;
import lee.joohan.daoexam.dto.Role;

public class JDBCTest {

	//DB Config에서 설정한 DataSource 객체를 가져와서 연결 부르기
	public static void main(String[] args) {
		ApplicationContext applicationContext = new AnnotationConfigApplicationContext(ApplicationConfig.class);
		RoleDao roleDao = applicationContext.getBean(RoleDao.class);
        Role role = new Role();

        role.setRoleId(500);
        role.setDescription("CEO");

		int count = roleDao.insert(role);

        System.out.println("Inserted: " + count);

       	role.setRoleId(201);
        role.setDescription("PROGRAMMER");

        int count = roleDao.update(role);

        System.out.println("updated: " + count);
	}
}
```

### 10. Select & Delete 구현하기

- Query 추가하기

RoleDaoSqls.java

```java
package lee.joohan.daoexam.dao;

public class RoleDaoSqls {
	public static final String SELECT_ALL = "SELECT role_id, description FROM role order by role_id";
    public static final String UPDATE = "UPDATE role set description = :description where role_id = :roleId";
    public static final String SELECT_BY_ROLE_ID = "SELECT role_id, description where role_id = :roleId";
    public static final String DELETE_BY_ROLE_ID = "DELETE FROM role where role_id = :roleId";
}
```

- RoleDao에 Method 추가

RoleDao.java

```java
package lee.joohan.daoexam.dao;

import java.util.Collections;
import java.util.List;

import javax.sql.DataSource;

import org.springframework.jdbc.core.BeanPropertyRowMapper;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate;
import org.springframework.stereotype.Repository;

import lee.joohan.daoexam.dto.Role;

@Repository
public class RoleDao {
	private NamedParameterJdbcTemplate namedParameterJdbcTemplate;
    private SimpleJdbcInsert insertAction;
	private RowMapper<Role> rowMapper = BeanPropertyRowMapper.newInstance(Role.class); 	//Java의  변수 표기방식과 DBMS의 Column name 표기 방식을 서로 호환해줌 ex) Java- roleId, DBMS - role_id

	public RoleDao(DataSource dataSource) {
		this.namedParameterJdbcTemplate = new NamedParameterJdbcTemplate(dataSource);
        this.insertAction = new SimpleJdbcInsert(dataSource)
            .withTableName("role");
	}

	public List<Role> selectAll() {
		return namedParameterJdbcTemplate.query(RoleDaoSqls.SELECT_ALL, Collections.emptyMap(), rowMapper); // 2nd Parameter: SQL에서 parameter에 바인딩 할 값들을 넘겨주는 목적
	}

    public int insert(Role role) {
        // 원래는 primary key를 자동으로 생성하도록 구현하지만 일단은 단순하게 primary key 직접 넣어주도록하자
        SqlParameterSource params = new BeanPropertySqlParameterSource(role);

        return insertAction.execute(params);
    }

    public int update(Role role) {
        SqlParameterSource params = new BeanPropertySqlParameterSource(role);

        return namedParameterJdbcTemplate.update(RoleDaoSqls.UPDATE, params)
    }

    public int deleteById(Integer id) {
        Map<String, ?> params = Collections.singletonMap("roleId", id);

        return namedParameterJdbcTemplate.update(RoleDaoSqls.DELETE_BY_ROLED_ID, params);
    }

    public Role selectById(Integer id) {
        try {
     		Map<String, ?> params = Collections.singletonMap("roleId", id);

            return namedParameterJdbcTemplate.queryForObject(RoleDaoSqls.SELECT_BY_ROLED_ID, params, rowMapper);
        } catch(EmptyResultDataAccessException e) {
            return null;
        }
    }    
}
```

- Test 해보기

JDBCTest.java

```java
package lee.joohan.daoexam;

import java.util.List;

import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

import lee.joohan.daoexam.config.ApplicationConfig;
import lee.joohan.daoexam.dao.RoleDao;
import lee.joohan.daoexam.dto.Role;

public class JDBCTest {

	//DB Config에서 설정한 DataSource 객체를 가져와서 연결 부르기
	public static void main(String[] args) {
		ApplicationContext applicationContext = new AnnotationConfigApplicationContext(ApplicationConfig.class);
		RoleDao roleDao = applicationContext.getBean(RoleDao.class);
        Role role = new Role();

        role.setRoleId(500);
        role.setDescription("CEO");

		int count = roleDao.insert(role);

        System.out.println("Inserted: " + count);

       	role.setRoleId(201);
        role.setDescription("PROGRAMMER");

        int count = roleDao.update(role);

        System.out.println("updated: " + count);

        Role selectedRole = roleDao.selectById(201);

        System.out.println("Selected role is: " + selectedRole);

        int deletedCount = roleDao.deleteById(500);
        System.out.println("Deleted: " + deletedCount);

        Role deletedRole = roleDao.selectById(201);

        System.out.println("Deleted role: " + deletedRole); // null


	}
}
```

TODO: Test 코드를 junit으로 구현하기

images from: https://www.edwith.org/boostcourse-web
원본 강의: https://www.edwith.org/boostcourse-web
