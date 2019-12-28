---
layout: post
title: "예제로 배우는 스프링 입문 (인프런 백기선님 강의) 정리 노트"
date: 2019-09-03 21:09:29 +0900
tags: ["spring", "java"]
categories: [spring]
---

## 1. Intro

- https://github.com/spring-projects/spring-petclinic.git 여기서 Clone하고 `./mvnw package`로 빌드하면됨
  - pom에서 package 세팅을 정하지 않으면 기본적으로 jar파일로 빌드됨
  - 필요에 따라 war 등으로 세팅할 수 있다
- 빌드 후 `java -jar target/*.jar` 명령어로 실행
  - 사실 안에 하나의 jar파일 밖에 없음
- http://localhost:8080/로 접속 후 페이지가 잘 생성되었는지 확인
- `./mvnw spring-boot:run` 으로 바로 빌드 후 실행되도록 할 수 있음

## 2. 프로젝트 살펴보기

-  resources/application.properties 에서 log level을 INFO에서 DEBUG로 수정

```properties
logging.level.org.springframework.web=DEBUG
```

- 웹페이지에서 어떤 행동을 했을때 어떤 controller로 맵핑되었는지 로그에 나타남
- Intellij의 Debugger를 이용해서 살펴볼 수도 있음

## 3. 코드 고쳐보기

- LastName이 아니라 FirstName으로 검색
  - 정확히 일치하는 것이 아니라 해당 키워드를 포함하면 리턴해주기
- Owner에 Age 추가

## 4. Inversion Of Control

- 내가 Dependency Injection을 하는게 아니라 다른 사람에게 넘긴다는 개념

- IOC Container = Bean Factory

  - ​	Bean을 만들고 엮어주며 제공해준다.

- ApplicationContext는 Bean Factory를 상속 받아서 다양한 일을 처리함

  - ApplicationContext에 직접 접근해서 Bean을 가져올 일을 거의 없다

- IOC Container안에 있는 객체끼리만 서로 Dependency Injection이 가능

  - Container 외부 객체에도 복잡한 방법으로 주입이 가능하긴 하지만 왠만해선 하지 않음

- Bean

  - IOC Container에 의해 관리되는 **객체**

  -  등록방법

    - ComponentScanning
      - 클래스에 @Component 태그를 붙인다
        - @Repository, @Service, @Controller 등등은 @Component를 Meta Annotation을 사용
      - Spring Boot에는 @ComponentScan이 자동 설정되어있어서 어디서 Bean을 스캔할지 설정안해도됨
    - 또는 직접 XML이나 Java Config에 등록

    ```java
    @Configuration
    public class SampleConfig {
        @Bean
        public SampleController sampleController() {
            return new SampleController();
        }
    }
    ```

  - Dependency Injection(의존성 주입 방법)

    - Setter와 Constructor를 이용해서 주입받을 수 있음

    - Contructor를 사용하길 권장하고 있음

    - ```java
      @Controller
      public class SampleController {
      	private final SampleRepository sampleRepository;

      	public SampleController(SampleRepository sampleRepository) {
          this.sampleRepository = sampleRepository
        }
      }
      ```

      - 필수적으로 사용해야할 Reference가 없으면 해당 instance를 만들지 못하도록 강제함
      - field에다가 @Autowired로 선언하면 내부 field의 Injection이 안된 상태로 해당 instance를 만들 수 있음

      ```java
      @Controller
      public class SampleController {
        @Autowired
        private SampleRepository sampleRepository; //SampleRepository가 비어있는 채로 SampleController가 생성됨
      }
      ```

      - 하지만, 객체가 서로 참조할 경우 Circular Injection 문제가 발생할 경우 setter를 사용하여 해결할 수 있음
        - 해당 문제가 발생하지 않도록 설계하는 것이 제일 좋지만 어쩔 수 없는 경우 setter를 사용

  ## 5. AOP (Aspect Oriented Programming)

  - 구현 방법
    - Compile
      - example.java가 example.class로 컴파일 될때 중간에 코드를 끼워넣어줌 (AspectJ에서 지원)
    - ByteCode 조작
      - 컴파일된 example.class를 실행할 때 classLoader가 해당 class를 메모리에 올릴 때 추가됨
    - 프록시 패턴
      - Spring AOP가 사용하는 방법
      - 기존의 코드를 건드리지 않고 다른 객체로 바꾸는 방법
  - ex) @Transactional



  ## 6. Proxy 구현 예제

  - Proxy란 기존 코드를 건드리지 않고 기능을 추가하는 것
  - Cash의 pay Method의 실행시간을 확인하기 위해 StopWatch 추가하기Payment

  ```java
  public interface Payment {
      void pay(int amount);
  }
  ```

  ```java
  public class Cash implements Payment{
      @Override
      public void pay(int amount) {
          System.out.println(amount + " paid by cash.");
      }
  }
  ```

  ```java
  public class CashPerf implements Payment {
      Payment cash = new Cash();

      @Override
      public void pay(int amount) {
          StopWatch stopWatch = new StopWatch();
          stopWatch.start();

          cash.pay(amount);

          stopWatch.stop();
          System.out.println(stopWatch.prettyPrint());
      }
  }
  ```

  ```java
  public class Store {
      private final Payment payment;

      public Store(Payment payment) {
          this.payment = payment;
      }

      public void buySomething(int amount) {
          payment.pay(amount);
      }
  }
  ```

  ```java
  public class StoreTest {

      @Test
      public void testPay() {
          Payment cashPerf = new CashPerf();
          Store store = new Store(cashPerf);

          store.buySomething(100);
      }
  }
  ```

  - CashPerf를 Proxy객체로 이용해서 Cash의 코드는 변경하지 않고 StopWatch를 추가



  ## 7. Spring AOP

  - Annotation을 이용 예제

    - @LogExecutionTime이라는 annotaion을 만들어서 사용

      Annotation class 생성

  ```java
  package org.springframework.samples.petclinic.sample;

  import java.lang.annotation.ElementType;
  import java.lang.annotation.Retention;
  import java.lang.annotation.RetentionPolicy;
  import java.lang.annotation.Target;

  @Target(ElementType.METHOD) //Method에만 적용할 수 있음
  @Retention(RetentionPolicy.RUNTIME) // Runtime까지 적용되도록
  public @interface LogExecutionTime {
  }

  ```

  위에서 생성한 Annotation에 사용될 Aspect 생성

  ```java
  package org.springframework.samples.petclinic.sample;

  import org.aspectj.lang.ProceedingJoinPoint;
  import org.aspectj.lang.annotation.Around;
  import org.aspectj.lang.annotation.Aspect;
  import org.slf4j.Logger;
  import org.slf4j.LoggerFactory;
  import org.springframework.stereotype.Component;
  import org.springframework.util.StopWatch;


  @Component
  @Aspect
  public class LogAspect {
      Logger logger = LoggerFactory.getLogger(LogAspect.class);

      @Around("@annotation(LogExecutionTime)") //LogExecutionTime annotation에서 작동하도록
      public Object logExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
          StopWatch stopWatch = new StopWatch();
          stopWatch.start();

          Object proceed = joinPoint.proceed();

          stopWatch.stop();
          logger.info(stopWatch.prettyPrint());

          return proceed;
      }
  }

  ```

  Controller에 Annotation 적용 (Controller에서 해당 Annotation이 적용된 method가 호출될 때마다 위의 Aspect가 작동됨)

  ```java
  @GetMapping("/owners/new")
  @LogExecutionTime
  public String initCreationForm(Map<String, Object> model) {
  	Owner owner = new Owner();

    model.put("owner", owner);

    return VIEWS_OWNER_CREATE_OR_UPDATE_FORM;
  }
  ```



  ## 8. Spring PSA (Portable Service Abstraction)

  - Servlet 방식의 코드 (Spring의 Controller의 뒤에서 일어나고 있음)

  ```java
  // /owner/create
  public class OwnerCreateServlet extends HttpServlet {

  //    GET
      @Override
      protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
          super.doGet(request, response);
      }

  //    POST
      @Override
      protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
          super.doPost(request, response);
      }
  }
  ```

  - Service Abstraction의 적용

  ```java
  @Controller
  public class SampleController {
  	...
  	@GetMapping("/owner/create")
  	public void create() {

  	}
  }
  ```

  - Spring MVC안에서의 Abstraction Layer

    - @Controller

      - PSA를 이용해서 Apache Netty적용하기 - 코드를 거의 바꾸지 않고도 가능
      - pom.xml의 dependency에 webflux추가

      ```xml
      <!-- <dependency> --> <!-- 기존 web dependency -->
      <!-- 	<groupId>org.springframework.boot</groupId> -->
      <!--   <artifactId>spring-boot-starter-web</artifactId> -->
      <!-- </dependency> -->
      <dependency>
      	<groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-webflux</artifactId>
      </dependency>
      ```

    - @Transactional

      - JDBC, JPA, Hibernate 등 다양하게 바꿔가며 사용가능

    ```java
    setAutocommit(false);

    commit();
    ```
    - @Cacheable
