---
title: "ServletBasic"
date: 2019-01-30T22:06:44+09:00
archives: "2019"
tags: ["edwith", "servlet"]
author: Joohan Lee
---

1. Java Web Application의 폴더 구조
  - WEB-INF가 반드시 존재해야됨
  - Servlet 3.0미만에선 web.xml이 필수적으로 있어야함. 이후 버전에는 annotation을 이용함

2. eclipse plugin
  - eclipse의 workspace안에 .metadata/.plugins/org.eclipse.wst.server.core/tmp0을 열어보면 Tomcat과 비슷 구조이다.
  - eclipse 내부적으로 Tomcat을 사용할 때 필요한 부분을 가져와서 쓴다고 생각하면 됨
  - wtpwebapps를 열어보면 여태까지 실습한 내용이 들어있음

3. Servlet이란?
  - Java Web Application의 구성요소 중 동적 처리를 맡음
  - WAS에서 동작하는 Java class이다.
  - HttpServlet class를 상속받아야한다
  - Servlet과 JSP로 최상의 결과를 얻으려면 조화롭게 사용해야 한다 (HTML은 JSP로, 복잡한 로직은 Servlet으로 처리)

4. Servlet 작동원리
  - Servlet 버전이 3.0 미만의 경우
  - Servlet이 생성될 때마다 web.xml에 등록을 해줘야됨
  - web.xml 파일의 <servlet-mapping>에서 해당 url이 존재하는지 찾아보고 없으면 404 있으면 Servlet으로 맵핑됨

ex)
```xml
<servlet>
    <description></description>
    <display-name>TenServlet</display-name>
    <servlet-name>TenServlet</servlet-name>
    <servlet-class>eaxmple.TenServlet</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>TenServlet</servlet-name>
    <url-pattern>/ten</url-pattern>
  </servlet-mapping>
```
3.0 이상의 경우엔 위의 과정이 아래 방식처럼 Annotation으로 처리됨

```java
@WebServlet("/ten")
```

5. Life Cycle of Servlet
  - Request가 왔을 때 WAS에서는 해당 Request를 처리할 Servlet이 있는지 검사
  - 없는 경우, Sevlet 객체를 생성 후 service가 불려짐(LifeCycleServlet() -> init() -> service())
  - 이미 객체가 존재하는 경우 service만 불림(service())
  - destroy의 경우 WAS가 종료되거나 Web Applicaiton이 새롭게 갱신될 경우 불려진다.
  - Servlet의 내용이 업데이트 된 경우 이미 생성된 Servlet 객체는 삭제되고 새롭게 생성됨 (destroy() -> LifeCycleServlet() -> init() -> service())

```java
@WebServlet("/LifecycleServlet")
public class LifecycleServlet extends HttpServlet {
	private static final long serialVersionUID = 1L;

	public LifecycleServlet() {
		//        super();
		System.out.println("Lifecycle Servlet is created.");
	}

	public void init(ServletConfig config) throws ServletException {
		System.out.println("init method is called.");
		System.out.println("init is changed."); // 먼저 요청을 보내 LifecycleServlet 객체가 생성된 이후에 해당 라인 추가
	}

	public void destroy() {
		System.out.println("destroy method is called.");
	}

	protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		System.out.println("service method is called.");
	}
}
```

6. service(request, response) 메소드
  - HttpServlet의 service 메소드는 템플릿 메소드 패턴으로 구현
  - 클라이언트의 요청이 GET일 경우 자신이 가지고 있는 doGet(request, response)를 호출
  - 클라이언트의 요청이 POST일 경우 자신이 가지고 있는 doPost(request, response)를 호출
  - PUT, DELETE도 마찬가

7. HttpServletRequest, HttpServletResponse
  - WAS는 웹 브라우저로부터 Servlet Request를 받으면,
  - 요청할 때 가지고 있는 정보를 HttpServletRequest 객체를 생성하여 저장
  - 웹 브라우저에게 응답을 보낼 때 사용하기 위하여 HttpServletResponse 객체를 생성
  - 생성된 HttpServletRequest, HttpServletResponse 객체를 Servlet에게 전달
  - HttpServletRequest
  - http protocol의 request 정보를 servlet에게 전달하기 위한 목적으로 사용
  - header, parameter, cookies, URI, URL 등의 정보를 읽어 들이는 method를 가지고 있다.
  - Body의 Stream을 읽어 들이는 method를 가지고 있다.
  - HttpServletResponse
  - WAS는 어떤 클라이언트가 요청을 보냈는지 알고 있고, 해당 클라이언트에게 응답을 보내기 위한 HttpServletResponse 객체를 생성해서 Servlet에게 전달
  - Servlet은 해당 객체를 이용해서 Content Type, Response code, Response message 등을 전송

8. forward
  - redirect의 경우 Client에서 Request를 두 번 보낸다.(302 reponse를 받고 다시 Request 보냄)
  - 하지만, forward의 경우엔 한 번의 Request와 Response만 존재함(Server안의 Servlet끼리 넘기는 방식이기 때문에)
  - 진행방식
    1. 웹 브라우저에서 Servlet1에게 요청을 보냄
    2. Servlet1은 요청을 처리한 후, 그 결과를 HttpServletRequest에 저장
    3. Servlet1은 결과가 저장된 HttpServletRequest와 응답을 위한 HttpServletResponse를 같은 웹 어플리케이션 안에 있는 Servlet2에게 전송(forward)
    4. Servlet2는 Servlet1으로 부터 받은 HttpServletRequest와 HttpServletResponse를 이용하여 요청을 처리한 후 웹 브라우저에게 결과를 전송

ex)
FrontServlet.java
```Java
package examples;

import java.io.IOException;

import javax.servlet.RequestDispatcher;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * Servlet implementation class FrontServlet
 */
@WebServlet("/front")
public class FrontServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;

    /**
     * @see HttpServlet#HttpServlet()
     */
    public FrontServlet() {
        super();
        // TODO Auto-generated constructor stub
    }

    /**
     * @see HttpServlet#service(HttpServletRequest request, HttpServletResponse response)
     */
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

            int diceValue = (int)(Math.random() * 6) + 1;
            request.setAttribute("dice", diceValue);

            RequestDispatcher requestDispatehcer = request.getRequestDispatcher("/next");
            requestDispatehcer.forward(request, response);
    }

}
```

NextServlet.java
```Java
package examples;

import java.io.IOException;
import java.io.PrintWriter;
import java.util.Enumeration;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * Servlet implementation class NextServlet
 */
@WebServlet("/next")
public class NextServlet extends HttpServlet {
	private static final long serialVersionUID = 1L;

	/**
	 * @see HttpServlet#HttpServlet()
	 */
	public NextServlet() {
		super();
		// TODO Auto-generated constructor stub
	}

	/**
	 * @see HttpServlet#service(HttpServletRequest request, HttpServletResponse response)
	 */
	protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		response.setContentType("text/html");
		PrintWriter out = response.getWriter();
		out.println("<html>");
		out.println("<head><title>form</title></head>");
		out.println("<body>");

		int dice = (Integer)request.getAttribute("dice");
		out.println("dice : " + dice);
		for(int i = 0; i < dice; i++) {
			out.print("<br>hello");
		}
		out.println("</body>");
		out.println("</html>");
	}
}

```
9. Servlet과 JSP 연동
  - 로직 수행은 Servlet에서, 화면 표시는 JSP에서 하는 것이 유리하기 때문에 Servlet에서 프로그램 로직을 수행하고, 그 결과를 JSP에게 forward처리

ex)
```Java
package examples;

import java.io.IOException;

import javax.servlet.RequestDispatcher;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * Servlet implementation class LogicServlet
 */
@WebServlet("/LogicServlet")
public class LogicServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;

    public LogicServlet() {
        super();
    }

    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        int v1 = (int)(Math.random() * 100) + 1;
        int v2 = (int)(Math.random() * 100) + 1;
        int result = v1 + v2;

        request.setAttribute("v1", v1);
        request.setAttribute("v2", v2);
        request.setAttribute("result", result);

        RequestDispatcher requestDispatcher = request.getRequestDispatcher("/result.jsp");
        requestDispatcher.forward(request, response);
    }
}
```
result.jsp
```java
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
EL표기법<br>
${v1} + ${v2} = ${result} <br><br>

Scriptlet과 Expression을 이용해 출력합니다.<br>
<%
    int v1 = (int)request.getAttribute("v1");
    int v2 = (int)request.getAttribute("v2");
    int result = (int)request.getAttribute("result");
%>

<%=v1%> + <%=v2 %> = <%=result %>
</body>
</html>
```

10. Scope 범위
  - Application scope: 웹 어플리케이션이 시작되고 종료될 때까지 변수가 유지되는 경우 사용
  - Session scope : 웹 브라우저 별로 변수가 관리되는 경우 사용
  - Request scope : http요청을 WAS가 받아서 웹 브라우저에게 응답할 때까지 변수가 유지되는 경우 사용
  - Page scope: 페이지 내에서 지역변수처럼 사용
  - 위 scope들의 사용 방법은 모두 같다.
      - 값을 저장할 때는 request 객체의 setAttribute()메소드를 사용한다.
      - 값을 읽어 들일 때는 request 객체의 getAttribute()메소드를 사용한다.

11. Page scope
  - 해당 Servlet이나 JSP이 실행되는 동안만 유지하려고 할 때 사용됨
  - PageContext라는 Abstract class를 이용
  - JSP 페이지 내에 pageContext라는 내장 객체 사용
  - forward할 경우 page scope 안의 context는 넘길 수 없다.
  - jsp에서 pageScope에 값을 저장한 후 해당 값을 EL표기법으로 사용할 때 자주 이용됨

12. Request scope

  - HTTP Request를 받아서 Response를 보낼 때까지 변수값을 유지하고자 할 경우 사용한다.
  - HttpServletRequest 객체를 사용한다.
  - JSP에서는 request 내장 변수를 사용한다.
  - 서블릿에서는 HttpServletRequest 객체를 사용한다.
  - forward 시 값을 유지하고자 사용한다.
  - 앞에서 forward에 대하여 배울 때 forward 하기 전에 request 객체의 setAttribute() 메소드로 값을 설정한 후, 서블릿이나 jsp에게 결과를 전달하여 값을 출력하도록 하였는데 이렇게 포워드 되는 동안 값이 유지되는 것이 Request scope를 이용했다고 합니다.

13. Session scope
  - Browser에 따라 변수를 관리하고자 할 경우 사용한다.
  - Browser의 Tab 간에는 세션정보가 공유되기 때문에, 각각의 탭에서는 같은 세션정보를 사용할 수 있다.
  - HttpSession 인터페이스를 구현한 객체를 사용한다.
  - JSP에서는 session 내장 변수를 사용한다.
  - 서블릿에서는 HttpServletRequest를 사용한다.
  - 장바구니처럼 사용자별로 유지가 되어야 할 정보가 있을 때 사용한다.

14. Application scope
  - Web Application이 시작되고 종료될 때까지 하나의 application객체가 사용된다.
  - ServletContext 인터페이스를 구현한 객체를 사용한다.
  - jsp에서는 application 내장 객체를 이용한다.
  - Servlet에서는 getServletContext()메소드를 이용하여 application객체를 이용한다.
  - 모든 클라이언트가 공통으로 사용해야 할 값들이 있을 때 사용한다.


ex)
ApplicationScope01.java
```java
package examples;

import java.io.IOException;
import java.io.PrintWriter;

import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@WebServlet("/ApplicationScope01")
public class ApplicationScope01 extends HttpServlet {
    private static final long serialVersionUID = 1L;

    public ApplicationScope01() {
        super();
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html; charset=UTF-8");

        PrintWriter out = response.getWriter();
        ServletContext application = getServletContext();
        int value = 1;

        application.setAttribute("value", value);
        out.println("<h1>value : " + value + "</h1>");
    }
}
```
ApplicationScope02.java
```Java
package examples;

import java.io.IOException;
import java.io.PrintWriter;

import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * Servlet implementation class ApplicationScope01
 */
@WebServlet("/ApplicationScope02")
public class ApplicationScope02 extends HttpServlet {
    private static final long serialVersionUID = 1L;

    /**
     * @see HttpServlet#HttpServlet()
     */
    public ApplicationScope02() {
        super();
        // TODO Auto-generated constructor stub
    }

    /**
     * @see HttpServlet#doGet(HttpServletRequest request, HttpServletResponse response)
     */
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html; charset=UTF-8");

        PrintWriter out = response.getWriter();

        ServletContext application = getServletContext();


        try {
            int value = (int)application.getAttribute("value");
            value++;
            application.setAttribute("value", value);
            out.println("<h1>value : " + value + "</h1>");
        }catch(NullPointerException ex) {
            out.println("value is not found.");
        }


    }

}
```
ApplicationScope01.jsp

```html
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
<%
    try{
        int value = (int)application.getAttribute("value");
        value = value + 2;
        application.setAttribute("value", value);
%>
        <h1><%=value %></h1>
<%        
    }catch(NullPointerException ex){
%>
        <h1>설정된 값이 없습니다.</h1>
<%        
    }
%>

</body>
</html>
```

원본 강의: https://www.edwith.org/boostcourse-web
