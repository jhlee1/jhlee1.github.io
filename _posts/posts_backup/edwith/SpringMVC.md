---
title: "Spring MVC"
date: 2019-04-29T12:43:40+09:00
archives: "2019"
tags: ["spring", "edwith", "webprogramming"]
author: Joohan Lee
---

## Spring MVC

### 1. MVC?
- Model-View-Controller의 약자
- 제록스 연구소에서 일하던 린즈커그가 처음 소개한 개념으로, Desktop Application용으로 고안됨
- Model: View가 렌더링하는데 필요한 Data
  - ex) 사용자가 요청한 상품 목록, 주문 내역
- View: 실제 보이는 부분, Model을 사용하여 렌더링 한다.
  - ex) jsp, jsf, pdf, xml등으로 결과를 표현
- Controller: 사용자의 액션에 응답하는 Componennt. Controller는 Model을 업데이트하고 다른 액션을 수행

### 2. Model 1 방식

- Jsp에 HTML과 Java코드가 섞여서 복잡해짐

![MVC_Model_1](..\..\..\resources\images\edwith\SpringMVC\MVC_Model_1.png)
### 3. Model 2 방식
![MVC_Model_2](..\..\..\resources\images\edwith\SpringMVC\MVC_Model_2.png)
### 4. Model 2의 발전형태

- 프론트 컨트롤러는 컨트롤러에게 요청을 위임

![MVC_Model_2_Improved](..\..\..\resources\images\edwith\SpringMVC\MVC_Model_2_Improved.png)
### 5. Spring Web Module

- Model 2 MVC 패턴을 지원하는 Spring Module

![Spring_Web_Module](..\..\..\resources\images\edwith\SpringMVC\Spring_Web_Module.png)
### 6. Spring MVC 기본 동작 흐름
1. Dispatcher Servlet이 요청을 받음
2. Dispatcher Servlet이 Handler Mapping에게 처리할 Controller와 Method를 물어봄 (Handler Mapping은 xml이나 configuration 클래스에서 정보를 가져와서 판단함)
3. Dispatcher Servlet이 Hanler Adapter에게 실행을 요청
4. 해당 Controller의 Method가 실행됨
5. Controller가 Handler Adapter를 통해서 view name을 보냄
6. 받은 View name를 View Resolver에게 넘기고
7. Dispatcher Servlet이 View Resolver에게 받은 View를 출력


![Spring_MVC_Flow](..\..\..\resources\images\edwith\SpringMVC\Spring_MVC_Flow.png)

### 7. DispatcherServlet

- Front Controller의 역할
- Client의 모든 요청을 받아서 적절한 Handler에게 넘기고 Handler에서 처리한 결과를 받아서 Client에게 Response를 보냄
- 여러 컴포넌트를 이용해서 작업 처리
- 내부 동작 흐름
  - 선처리 후 HandlerExecutionChain 실행
    - Exception 발생 ->  Exception 처리
    - 정상처리 -> View 렌더링 후 Response로 보내줌

![DispatcherServlet_Flow](..\..\..\resources\images\edwith\SpringMVC\DispatcherServlet_Flow.png)

- 요청 선처리 작업 내부
  - Locale 결정 - 브라우저 언어 설정을 이용해 알 수 있다
    - **org.springframework.web.servlet.LocaleResolver**
    - 지역 정보를 결정해주는 전략 오브젝트
    - 디폴트인 AcceptHeaderLocalResolver는 HTTP 헤더의 정보를 보고 지역정보를 설정
  - RequestContextHolder 
    - **org.springframework.web.context.request.RequestContextHolder**
    - 일반 빈에서 HttpServletRequest, HttpServletResponse, HttpSession 등을 사용할 수 있도록 한다.
    - 해당 객체를 일반 빈에서 사용하게 되면, Web에 종속적이 될 수 있다.
  - 
    - **org.springframework.web.servlet.FlashMapManager**
    - FlashMap객체를 조회(retrieve) & 저장을 위한 인터페이스
    - RedirectAttributes의 addFlashAttribute메소드를 이용해서 저장
    - 리다이렉트 후 조회를 하면 바로 정보는 삭제
  - FlashMap - Redirect될 때 값을 전달하기 위해 쓰임
    - **org.springframework.web.servlet.FlashMapManager**
    - FlashMap객체를 조회(retrieve) & 저장을 위한 인터페이스
    - RedirectAttributes의 addFlashAttribute메소드를 이용해서 저장
    - 리다이렉트 후 조회를 하면 바로 정보는 삭제
  - 멀티파트 - 파일 업로드의 경우에 사용됨
    - **org.springframework.web.multipart.MultipartResolver**

![PreProcessing_FLow](..\..\..\resources\images\edwith\SpringMVC\PreProcessing_FLow.png)

- Handler처리 관련 내부 동작 흐름

  - **org.springframework.web.servlet.HandlerMapping**
    - HandlerMapping구현체는 어떤 핸들러가 요청을 처리할지에 대한 정보를 알고 있다.
    - 디폴트로 설정되는 있는 핸들러매핑은 BeanNameHandlerMapping과 DefaultAnnotationHandlerMapping 2가지가 설정되어 있다.
  - org.springframework.web.servlet.HandlerExecutionChain
    - HandlerExecutionChain구현체는 실제로 호출된 핸들러에 대한 참조를 가지고 있다.
    - 즉, 무엇이 실행되어야 될지 알고 있는 객체라고 말할 수 있으며, 핸들러 실행전과 실행후에 수행될 HandlerInterceptor도 참조하고 있다.

  - org.springframework.web.servlet.HandlerAdapter
    - 실제 핸들러를 실행하는 역할을 담당한다.
    - 핸들러 어댑터는 선택된 핸들러를 실행하는 방법과 응답을 ModelAndView로 변화하는 방법에 대해 알고 있다.
    - 디폴트로 설정되어 있는 핸들러어댑터는 HttpRequestHandlerAdapter, SimpleControllerHandlerAdapter, AnnotationMethodHanlderAdapter 3가지이다.
    - @RequestMapping과 @Controller 애노테이션을 통해 정의되는 컨트롤러의 경우 DefaultAnnotationHandlerMapping에 의해 핸들러가 결정되고, 그에 대응되는 AnnotationMethodHandlerAdapter에 의해 호출이 일어난다.



![DispatcherServlet_Handler_Flow](..\..\..\resources\images\edwith\SpringMVC\DispatcherServlet_Handler_Flow.png)

- 요청 처리 과정과  Intercepter
  - **org.springframework.web.servlet.ModelAndView**
    - ModelAndView는 Controller의 처리 결과를 보여줄 view와 view에서 사용할 값을 전달하는 클래스이다.
  - **org.springframework.web.servlet.RequestToViewNameTranslator**
    - 컨트롤러에서 뷰 이름이나 뷰 오브젝트를 제공해주지 않았을 경우 URL과 같은 요청정보를 참고해서 자동으로 뷰 이름을 생성해주는 전략 오브젝트이다. 디폴트는 DefaultRequestToViewNameTranslator이다.

![DispatcherServlet_Intercepter_Flow](..\..\..\resources\images\edwith\SpringMVC\DispatcherServlet_Intercepter_Flow.png)

- Exception 처리
  - **org.springframework.web.servlet.handlerexceptionresolver**
    - 기본적으로 DispatcherServlet이 DefaultHandlerExceptionResolver를 등록한다.
    - HandlerExceptionResolver는 예외가 던져졌을 때 어떤 핸들러를 실행할 것인지에 대한 정보를 제공한다.

![DispatcherServlet_Exception_Flow](..\..\..\resources\images\edwith\SpringMVC\DispatcherServlet_Exception_Flow.png)

- View rendering

  - **org.springframework.web.servlet.ViewResolver**
    - 컨트롤러가 리턴한 뷰 이름을 참고해서 적절한 뷰 오브젝트를 찾아주는 로직을 가진 전략 오프젝트이다.
    - 뷰의 종류에 따라 적절한 뷰 리졸버를 추가로 설정해줄 수 있다.

  ![DispatcherServlet_ViewRendering_Flow](..\..\..\resources\images\edwith\SpringMVC\DispatcherServlet_ViewRendering_Flow.png)

- 요청 처리 완료

![DispatcherServlet_RequestClose_Flow](..\..\..\resources\images\edwith\SpringMVC\DispatcherServlet_RequestClose_Flow.png)








원본 강의: https://www.edwith.org/boostcourse-web
