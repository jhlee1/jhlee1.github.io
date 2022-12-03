---
title: "Spring MVC Implementation"
date: 2019-05-05T06:59:02+09:00
archives: "2019"
tags: ["spring", "edwith", "webprogramming"]
author: Joohan Lee
---

## Spring MVC 실습

### 1. Controller 작성 실습

- 브라우저에서 `http://localhost:8080/mvcexam/plusform`이라고 요청을 보내면 서버는 2개의 값을 입력받을 수 있는 입력 창과 버튼이 있는 화면을 보여줌
- 2개의 값을 입력하고 버튼을 클릭하면 `POST http://localhost:8080/mvcexam/plus`로 2개의 입력값이 전달됨. 서버는 두 값을 더한 후 jsp에게 request scope로 전달하여 출력



### 2. 프로젝트 세팅

- maven 프로젝트에서 archetype을 webapp으로 생성
- pom.xml 설정
  - Dependency와 Plugin에 Maven compiler, JSTL, Servlet JSP, Spring 관련 설정들을 넣어준다.

```
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
    <spring.version>4.3.5.RELEASE</spring.version>
  </properties>

  <dependencies>
    ...

    <!-- Spring Spring MVC -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>${spring.version}</version>
    </dependency>

    <!-- Servlet JSP JSTL -->
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>3.1.0</version>
      <scope>provided</scope>
    </dependency>
    <dependency>
      <groupId>javax.servlet.jsp</groupId>
      <artifactId>javax.servlet.jsp-api</artifactId>
      <version>2.3.1</version>
      <scope>provided</scope>
    </dependency>
    <dependency>
      <groupId>jstl</groupId>
      <artifactId>jstl</artifactId>
      <version>1.2</version>
    </dependency>
  </dependencies>
  <build>
    <finalName>mvcexam</finalName>
    <pluginManagement>
      <plugins>
        ...

        <plugin>
          <groupId>org.apache.maven.plugins</groupId>
          <artifactId>maven-compiler-plugin</artifactId>
          <version>3.6.1</version>
          <configuration>
            <source>1.8</source>
            <target>1.8</target>
          </configuration>
        </plugin>

        ...

      </plugins>
    </pluginManagement>
  </build>
</project>
```

### 3. DispatcherServlet을 FrontController로 설정

- 3가지 방법이 존재한다. 주로 1번과 2번 방법을 사용

  1. web.xml 파일에 설정

  2. org.springframework.web.WebApplicationInitializer 인터페이스를 이용해서 구현

  3. javax.servlet.ServletContainerInitializer 사용
     - Servlet 3.0 이상에서 web.xml 파일 대신 사용 가능

- web.xml 파일에 설정 (1번 방법)
  - xml 파일과 java파일을 이용해서  DispatcherServlet을 설정할 수 있다.

ex) WebMVCContextConfig.xml에서 설정을 읽어들이는 방법

```xml
<?xml version="1.0" encoding = "UTF-8"?>
<web-app>
  <servlet>
    <servlet-name>dispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:WebMVCContextConfig.xml</param-value>
    </init-param>
  </servlet>
</web-app>
```

 ex) java config spring 설정을 읽어들이는 방법

```xml
<?xml version="1.0" encoding = "UTF-8"?>
<web-app>
  <display-name>Spring JavaConfig Example</display-name>
  <servlet>
    <servlet-name>mvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <param-name>contextClass</param-name>
      <param-value>org.springframework.web.context.support.AnnotationConfigWebApplicationContext</param-value>
    </init-param>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>lee.joohan.webmvc.config.WebMvcContextConfiguration</param-value>
    </init-param>
  </servlet>
  <servlet-mapping>
    <servlet-name>mvc</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
</web-app>
```

- org.springframework.web.WebApplicationInitializer 인터페이스를 이용해서 구현 (2번 방법)
  - Spring MVC는 ServletContainerInitializer를 구현하고 있는 SpringServletContainerInitializer를 제공
  - SpringServletContainerInitializer는 WebApplicationInitializer 구현체를 찾아서 객체를 만들고 해당 객체의 onStartup을 호출하여 초기화 (첫 웹앱 시작에 다소 시간이 걸림)

ex)

```java
import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.ServletRegistration;

import org.springframework.web.WebApplicationInitializer;
import org.springframework.web.context.support.AnnotationConfigWebApplicationContext;
import org.springframework.web.servlet.DispatcherServlet;

public class SpringWebApplicationInitializer implements WebApplicationInitializer {
    public static final String DISPATCHER_SERVLET_NAME = "dispatcher";

    @Override
    public void onStartup(ServletContext servletContext) throws ServletException {
        registerDispatcherServlet(servletContext);
    }

    private void registerDispatcherServlet(ServletContext servletContext) {
        AnnotationConfigWebApplicationContext dispatcherContext = createContext(WebMVCContextConfiguration.class); //WebMVCContextConfiguration.class라는 java config 파일은 아래에
        ServletRegistration.Dynamic dispatcher = servletContext.addServlet(DISPATCHER_SERVLET_NAME, new DispatcherServlet(dispatcherContext));

        dispatcher.setLoadOnStartup(1);
        dispatcher.addMapping("/");
    }

    private AnnotationConfigWebApplicationContext createContext(final Class<?> annotatedClasses) {
        AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();

        context.register(annotatedClasses);
        return context;
    }
}

```

WebMvcContextConfiguration.java

```
package lee.joohan.webmvc.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.*;
import org.springframework.web.servlet.view.InternalResourceViewResolver;

@Configuration
@EnableWebMvc
@ComponentScan(basePackages = {"lee.joohan.webmvc.controller"})
public class WebMvcContextConfiguration extends WebMvcConfigurerAdapter {

    //모든 Request를 Controller에게 Mapping 되어있지 않은 CSS와 image 같은 요청의 경우 따로 처리
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/assets/**").addResourceLocations("classpath:/META-INF/resources/webjars/").setCachePeriod(31556926);
        registry.addResourceHandler("/css/**").addResourceLocations("/css/").setCachePeriod(31556926);
        registry.addResourceHandler("/img/**").addResourceLocations("/img/").setCachePeriod(31556926);
        registry.addResourceHandler("/js/**").addResourceLocations("/js/").setCachePeriod(31556926);
    }

    // Default servlet handler를 사용 (Mapping정보가 없는 URL의 경우 WAS에게 넘겨 DefaultServlet을 통해 처리하도록
    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }

    @Override
    public void addViewControllers(final ViewControllerRegistry registry) {
        registry.addViewController("/").setViewName("main"); // 특정 url의 경우 따로 controller를 거치지 않고 view를 보여주도록 설정
    }

    @Bean
    public InternalResourceViewResolver getInternalResourceViewResolver() {
        InternalResourceViewResolver resolver = new InternalResourceViewResolver();

        resolver.setPrefix("/WEB-INF/views/");
        resolver.setSuffix(".jsp");

        return resolver;
    }
}

```

- @Configuration
  - Java Config 파일인 것을 알려줌
  - org.springframework.context.annotation의 Configuration Annotation과 Bean Annotation 코드를 이용해서 Spring Container에 새로운 Bean 객체 생성

- @EnableWebMvc
  - DispatcherServlet의 RequestMappingHandlerMapping, RequestMappingHandlerAdapter, ExceptionHandlerExceptionResolver, MessageConverter 등 Web에 필요한 빈들을 대부분 자동으로 설정해준다.
  - xml 설정에서 <mvc:annotation-driven/> 와 동일하다.
  - 기본 설정 이외의 설정이 필요하다면 org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter 를 상속받도록 Java config class를 작성한 후, 필요한 메소드를 @Override 하면됨

- @ComponentScan
  - Controller, Service, Repository, Component애노테이션이 붙은 클래스를 찾아 스프링 컨테이너가 관리하게 된다.
  - DefaultAnnotationHandlerMapping과 RequestMappingHandlerMapping구현체는 다른 Handler Mapping보다 훨씬 더 정교한 작업을 수행한다. 
    - Spring Container(= AppicationContext)에 있는 요청 처리 Bean에서 RequestMapping Annotation을 클래스나 메소드에서 찾아 HandlerMapping객체를 생성하게 된다.
  - HandlerMapping은 서버로 들어온 요청을 어느 핸들러로 전달할지 결정하는 역할을 수행한다.
  - DefaultAnnotationHandlerMapping은 DispatcherServlet이 기본으로 등록하는 기본 핸들러 맵핑 객체이고, RequestMappingHandlerMapping은 더 강력하고 유연하지만 사용하려면 명시적으로 설정해야 한다



### 4. Controller(Handler) 클래스 만들기

- @Controller Annotation을 클래스 위에 붙인다
- Mapping을 위해 @RequestMapping Annotation을 클래스나 메소드에서 사용
- @RequestMapping
  - Http 요청과 이를 다루기 위한 Controller 의 메소드를 연결하는 Annotation
  - Http Method 와 연결하는 방법
    - @RequestMapping(value="/users", method=RequestMethod.POST)
    - Spring 4.3 이상에선 아래와 같이 사용 가능
      - @GetMapping
      - @PostMapping
      - @PutMapping
      - @DeleteMapping
      - @PatchMapping
  - Http 특정 해더와 연결하는 방법
    - @RequestMapping(method = RequestMethod.GET, headers = "content-type=application/json")
  - Http Parameter 와 연결하는 방법
    - @RequestMapping(method = RequestMethod.GET, params = "type=raw")
  - Content-Type Header 와 연결하는 방법
    - @RequestMapping(method = RequestMethod.GET, consumes = "application/json")
  - Accept Header 와 연결하는 방법
    - @RequestMapping(method = RequestMethod.GET, produces = "application/json")

- Spring MVC가 지원하는 Controller메소드 인수 타입
  - javax.servlet.ServletRequest
  - **javax.servlet.http.HttpServletRequest**
  - org.springframework.web.multipart.MultipartRequest
  - org.springframework.web.multipart.MultipartHttpServletRequest
  - javax.servlet.ServletResponse
  - **javax.servlet.http.HttpServletResponse**
  - **javax.servlet.http.HttpSession**
  - org.springframework.web.context.request.WebRequest
  - org.springframework.web.context.request.NativeWebRequest
  - java.util.Locale
  - java.io.InputStream
  - java.io.Reader
  - java.io.OutputStream
  - java.io.Writer
  - javax.security.Principal
  - java.util.Map
  - org.springframework.ui.Model
  - org.springframework.ui.ModelMap
  - **org.springframework.web.multipart.MultipartFile**
  - javax.servlet.http.Part
  - org.springframework.web.servlet.mvc.support.RedirectAttributes
  - org.springframework.validation.Errors
  - org.springframework.validation.BindingResult
  - org.springframework.web.bind.support.SessionStatus
  - org.springframework.web.util.UriComponentsBuilder
  - org.springframework.http.HttpEntity<?>
  - Command 또는 Form 객체

- **Spring MVC가 지원하는 메소드 인수 애노테이션**
  - **@RequestParam**
    - Mapping된 메소드의 Argument에 붙일 수 있는 어노테이션
    - @RequestParam의 name에는 http parameter의 name과 멥핑
    - @RequestParam의 required는 필수인지 아닌지 판단
  - **@RequestHeader**
    - 요청 정보의 헤더 정보를 읽어들 일 때 사용@RequestHeader(name="헤더명") String 변수명
  - **@RequestBody**
  - @RequestPart
  - **@ModelAttribute**
  - **@PathVariable**
    - @RequestMapping의 path에 변수명을 입력받기 위한 place holder가 필요함
    - place holder의 이름과 PathVariable의 name 값과 같으면 mapping 됨
    - required 속성은 default true 임
  - @CookieValue

 요

 

- **Spring MVC가 지원하는 메소드 리턴 값**
  - **org.springframework.web.servlet.ModelAndView**
  - org.springframework.ui.Model
  - java.util.Map
  - org.springframework.ui.ModelMap
  - org.springframework.web.servlet.View
  - **java.lang.String**
  - java.lang.Void
  - org.springframework.http.HttpEntity<?>
  - org.springframework.http.ResponseEntity<?>
  - 등등

**PlusController.java**

```java
package lee.joohan.webmvc.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.ModelMap;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;

@Controller
public class PlusController {
    @GetMapping(path = "/plusform")
    public String getPlusForm() {
        return "plusForm";
    }

    @PostMapping(path = "/plus")
    public String plus(@RequestParam(name = "value1", required = true) int value1,
                       @RequestParam(name = "value2", required = true) int value2, ModelMap modelMap) {
        modelMap.addAttribute("value1", value1);
        modelMap.addAttribute("value2", value2);
        modelMap.addAttribute("result", value1 + value2);

        return "plusResult";
    }
}

```

**plusForm.jsp**

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" pageEncoding="UTF-8" %>
<html>
<head>
    <title>Plus form</title>
</head>
<body>
    <form method="POST" action="plus">
        value1: <input type="text" name="value1"><br/>
        value2: <input type="text" name="value2"><br/>
        <input type="submit" value="확인">
    </form>
</body>
</html>

```

**plusResult.jsp**

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Plus Result</title>
</head>
<body>
    ${value1} + ${value2} = ${result}
</body>
</html>
```

### 5. 추가 실습

- 기능 추가 1
  - http://localhost:8080/mvcexam/userform 으로 요청을 보내면 이름, email, 나이를 물어보는 폼이 보여진다.
  - 폼에서 값을 입력하고 확인을 누르면 post방식으로 http://localhost:8080/mvcexam/regist 에 정보를 전달하게 된다.
  - regist에서는 입력받은 결과를 콘솔 화면에 출력한다.

**UserController.java**

```java
package lee.joohan.webmvc.controller;

import lee.joohan.webmvc.dto.User;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.PostMapping;

@Controller
public class UserController {
    @GetMapping(path = "/userform")
    public String getUserForm() {
        return "UserForm";
    }

    @PostMapping(path = "/regist")
    public String regist(@ModelAttribute User user) { // Model을 바로 받기
        System.out.println("The user from input: " + user);

        return "Regist";
    }
}

```

**UserForm.jsp**

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>User Form</title>
</head>
<body>
<form method="post", action="regist">
    name: <input type="text" name="name"><br/>
    email: <input type="text" name="email"><br/>
    age: <input type="text" name="age"><br/>
    <input type="submit" value="확인">
</form>
</body>
</html>

```

**Regist.jsp**

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Registered</title>
</head>
<body>
<h1>Registered</h1>
</body>
</html>

```

**User.java (DTO)**

```java
package lee.joohan.webmvc.dto;

public class User {
    private String name;

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", email='" + email + '\'' +
                ", age=" + age +
                '}';
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    private String email;
    private int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}

```

- 기능 추가 2
  - http://localhost:8080/mvcexam/goods/{id} 으로 요청을 보낸다.
  - 서버는 id를 콘솔에 출력하고, 사용자의 브라우저 정보를 콘솔에 출력한다.
  - 서버는 HttpServletRequest를 이용해서 사용자가 요청한 PATH정보를 콘솔에 출력한다.

**GoodsController.java**

```java
package lee.joohan.webmvc.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.ModelMap;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestHeader;

import javax.servlet.http.HttpServletRequest;

@Controller
public class GoodsController {
    @GetMapping("/goods/{id}")
    public String getGoodsById(@PathVariable(name = "id") int id, @RequestHeader(value = "User-Agent", defaultValue = "myBrowser") String userAgent,
                               HttpServletRequest request, ModelMap model) {
        String path = request.getServletPath();

        System.out.println("id: " + id);
        System.out.println("user_agent: " + userAgent);
        System.out.println("path: " + path);

        model.addAttribute("id", id);
        model.addAttribute("userAgent", userAgent);
        model.addAttribute("path", path);

        return "GoodsById";
    }
}

```

**GoodsById.jsp**

```
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Goods by Ids</title>
</head>
<body>
id: ${id} <br/>
user_agent: ${userAgent} <br/>
path: ${path} <br/>
</body>
</html>
```

