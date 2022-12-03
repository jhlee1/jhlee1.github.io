---
title: "SpringCore"
date: 2019-02-27T23:10:04+09:00
archives: "2019"
tags: ["spring", "edwith", "webProgramming"]
author: Joohan Lee
---

## Spring Core

### 1. Spring 특징

- 엔터프라이즈급 어플리케이션을 구축할 수 있는 가벼운 솔루션이자, One-Stop-Shop(모든 과정을 한꺼번에 해결)
- 모듈화
  - 약 20개의 모듈로 구성
  - 필요한 모듈만 가져다 사용
- IoC Container
- 선언적 Transaction을 관리할 수 있다
- MVC Framework 제공
- AOP

### 2. Spring Core 모듈

- AOP와 Instrumentation
  - spring-AOP : AOP 얼라이언스(Alliance)와 호환되는 방법으로 AOP를 지원
  - spring-aspects : AspectJ와의 통합을 제공
  - spring-instrument  : instrumentation을 지원하는 클래스와 특정 WAS에서 사용하는 클래스로 더 구현체를 제공
    BCI(Byte  Code Instrumentations) - 런타임이나 로드(Load) 때 클래스의 바이트 코드에 변경을 가하는 방법

-  Messaging

  - spring-messaging : 스프링 프레임워크 4부터 메시지 기반 어플리케이션을 작성할 수 있는 Message,  MessageChannel, MessageHandler 등을 제공

- 데이터 엑서스(Data Access) / 통합(Integration)
  - JDBC, ORM, OXM, JMS 및 트랜잭션 모듈로 구성
  - spring-jdbc : 자바 JDBC프로그래밍을 쉽게 할 수 있는 기능
  - spring-tx : 선언적 트랜잭션 관리를 할 수 있는 기능
  - spring-orm : JPA, JDO및 Hibernate를 포함한 ORM API를 위한 통합 레이어를 제공
  - spring-oxm : JAXB, Castor, XMLBeans, JiBX 및 XStream과 같은 Object/XML 맵핑
  - spring-jms : 메시지 생성(producing) 및 사용(consuming)을 위한 기능을 제공
    Spring Framework 4.1부터 spring-messaging모듈과의 통합

- 웹(Web)
  - spring-web, spring-webmvc, spring-websocket, spring-webmvc-portlet 모듈로 구성
    - spring-web : 멀티 파트 파일 업로드, 서블릿 리스너 등 웹 지향 통합 기능을 제공한다. HTTP클라이언트와 Spring의 원격 지원을 위한 웹 관련 부분을 제공
    - spring-webmvc : Web-Servlet 모듈이라고도 불리며, Spring MVC 및 REST 웹 서비스 구현을 포함
    - spring-websocket : 웹 소켓 지원
    - spring-webmvc-portlet : 포틀릿 환경에서 사용할 MVC 구현

### 3. IoC/DI Container

- Container
  - 인스턴스의 생명주기(LifeCycle) 관리
  - 생성된 인스턴스에 추가적인 기능 제공
  - ex) WAS가 Servlet을 관리해줌
- IOC(Inversion of Controller)
  - 제어의 역전 - 프로그램의 흐름을 개발자가 아니라 다른 프로그램이 하는 것
  - Factory Design Pattern 사용
- DI(Dependency Injection)
  - class 사이의 의존 관계를 Bean설정 정보를 바탕으로 container가 자동으로 연결해주는 것

ex) DI가 적용안된 경우

```java
class Engine {
}

class Car {
    Engine v5 = new Engine();
}
```

ex) DI 적용

```java
@Component
class Engine {
}

@Component
class Car {
    @Autowired
    Engine v5
}
```

- Spring에서 제공하는 IoC/DI Container
  - BeanFactory: IoC/DI에 대한 기본 기능 제공
  - ApplicationContext: BeanFactory의 모든 기능을 포함하며 Transaction처리, AOP 등에 대한 처리를 할 수 있다. BeanPostProcessor, BeanFactoryPostProcessor등을 자동으로 등록하고, 국제화 처리, 어플리케이션 이벤트 등을 처리
    - BeanPostProcessor: 컨테이너의 기본로직을  Overriding하여 인스턴스화 와 의존성 처리 로직 등을 개발자가 원하는 대로 구현 할 수 있다.
    - BeanFactoryPostProcessor: meta data를 커스터마이징 할 수 있다.
  - 일반적으로 BeanFactory는 너무 기본적인 기능만 제공하기 때문에 ApplicationContext를 더 많이 씀

### 4. IoC/DI 실습 - XML 파일을 이용한 설정

- Spring을 이용해서 Bean 객체 만들기
- Bean - Spring컨테이너가 관리하는 객체 (직접 new연산자로 생성해서 사용하는 객체는 빈(Bean)이라고 말하지 않음)
- Bean 생성 - 아래의 세가지 조건을 만족시켜야함
  - 기본 생성자를 가지고 있다
  - 필드는 private하게 선언
  - getter, setter를 가진다. getName(), setName() 메소드를 name property라고 한다. (용어 중요)

```java
public class UserBean {
	private String name;
	private int age;
	public UserBean() {

	}

	public UserBean(String name, int age, boolean male) {
		this.name = name;
		this.age = age;
		this.male = male;
	}

	private boolean male;

	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public int getAge() {
		return age;
	}
	public void setAge(int age) {
		this.age = age;
	}
	public boolean isMale() {
		return male;
	}
	public void setMale(boolean male) {
		this.male = male;
	}
}
```

- pom.xml 파일에서 <properties> 이용하여 dependency 추가하기

```xml
  <properties>
  	...
    <spring.version>4.3.14.RELEASE</spring.version>
    ...
  </properties>

  <dependencies>
  	<dependency>
  		<groupId>org.springframework</groupId>
  		<artifactId>spring-context</artifactId>
  		<version>${spring.version}</version>
  	</dependency>
  	...
  </dependencies>
```

- main/resources에 applicationContext.xml 파일 추가하기
- <bean> 태그 이용해서 UserBean 객체 생성하기

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
	<bean id="userBean" class="lee.joohan.diexam01.UserBean"></bean>
</beans>
```

### 5. XML 이용해서 Dependency Injection

- Engine class 생성

```
public class Engine {
	public Engine() {
		System.out.println("Engine constructor!");
	}

	public void exec() {
		System.out.println("Engine is running");
	}
}
```

- Engine class에 dependency를 가지는 Car 생성

```java
public class Car {
	private Engine v8;

	public Car() {
		System.out.println("Car constructor!");
	}

	public void setEngine(Engine e) {
		this.v8 = e;
	}

	public void run() {
		System.out.println("The car is running");
		v8.exec();
	}

	public static void main(String[] args) {
		Engine e = new Engine();
		Car c = new Car();

		c.setEngine(e);
		c.run();
	}
}
```

- applicationContext.xml에 bean 추가

```xml
<bean id="engine" class="lee.joohan.diexam01.Engine" />
<bean id="car" class="lee.joohan.diexam01.Car">
		<property name="engine" ref="engine"></property>
</bean>
```

- ApplicationContext를 생성하여 Spring Container에서 Car 객체 가져오기

```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class ApplicationContextExam02 {
	public static void main(String[] args) {
		ApplicationContext applicationContext = new ClassPathXmlApplicationContext("classpath:applicationContext.xml");
		Car car = (Car) applicationContext.getBean("car");

		car.run();
	}
}
```

### 6. IoC/DI 실습 - Java Config을 이용한 설정

#### 1. Annotations

- @Configuration

  - 스프링 설정 클래스를 선언하는 어노테이션

  @Bean

  - bean을 정의하는 어노테이션

  - ```
    @Autowired를 사용하지 않고 Bean 을 이용하는 이유: 외부 라이브러리(JDBC 등)에는 @Component와 같은 Annotation이 붙어있지 않기 때문에 이런식으로 생성해줘야함.
    ```

- @ComponentScan

  - 지정한 패키지 내에 있는 @Controller, @Service, @Repository, @Component 어노테이션이 붙은 클래스를 찾아 컨테이너에 등록

- @Component

  - 컴포넌트 스캔의 대상이 되는 애노테이션 중 하나로써 주로 유틸, 기타 지원 클래스에 붙이는 어노테이션

- @Autowired

  - 주입 대상이되는 bean을 컨테이너에 찾아 주입하는 어노테이션

#### 2. @Bean을 이용해 객체 생성하기

- @Bean 태그를 붙인 메소드 안에서 객체를 생성해서 return처리
- 메소드 이름이 bean id가 됨

ApplicationConfig.java

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class ApplicationConfig {
    @Bean
    public Car car(Engine e) {
        Car c = new Car();
		c.setEngine(e);
        return c;
    }

    @Bean
    public Engine engine() {
        return new Engine();
    }
}

```

- ApplicationContextExam03.java
- **Bean id**를 이용해서 객체를 가져올 수 있고
- Bean의 **class type**을 이용해서 가져올 수 있다.

ApplicationContextExam03.java

```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class ApplicationContextExam03 {
    public static  void main(String[] args) {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(ApplicationConfig.class);


//        Car car = (Car) applicationContext.getBean("car"); // 1번 방식
        Car car = applicationContext.getBean(Car.class); //2번 방식

        car.run();
    }
}
```

#### 3. @Component와 @Autowired를 이용해서 객체 생성하기

- @componentScan에 어떤 package에 있는 Annotation이 붙은 객체를 생성할지 알려줘야함

ApplicationConfig2.java

```java
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan("lee.joohan")
public class ApplicationConfig2 {
}
```

- 기존 Car와 Engine에 @Component 추가
- @Autowired를 이용하기 때문에 Car에서 setEngine이 필요 없음

Engine.java

```java
import org.springframework.stereotype.Component;

@Component
public class Engine {
    public Engine() {
        System.out.println("Engine constructor!");
    }

    public void exec() {
        System.out.println("Engine is being operated");
    }
}
```

Car.java

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class Car {
    @Autowired
    private Engine v8;

//    public void setEngine(Engine engine) {
//        v8 = engine;
//    }

    public Car() {
        System.out.println("Car contructor!");
    }

    public void run() {
        System.out.println("the car is running");
        v8.exec();
    }
}
```

- **ApplicationContextExam03.java**에서 부르는 방식 @Bean을 사용했을 때랑 똑같음.




원본 강의: https://www.edwith.org/boostcourse-web
