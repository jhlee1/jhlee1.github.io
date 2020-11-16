---
title: "JspBasic"
date: 2019-02-06T02:51:19+09:00
archives: "2019"
tags: ["edwith", "jsp", "webProgramming"]
author: Joohan Lee
---

1. JSP(Java Server Page)
   - 모든 JSP는 Servlet으로 변경되어 작동한다. -> Servlet과 같은 LifeCycle을 가짐
   - <%@를 지시자라고 한다. <%@ page는 페이지 지시자라고 한다.
   - JSP를 작성할 때 Servlet으로 어떻게 변경될까에 대해 생각하면서 하자
   - 1 ~ 10까지 출력하는 예제

```java
sum10.jsp

<%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="UTF-8" %>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Insert title here</title>
	</head>
    <body>
        <%
        	int total = 0;

            for(int i = 1; i <= 10; i++) {
                total = total + i;
            }
        %>

	1부터 10까지의 합: <%= total %>        
    </body>
</html>
```
2. JSP의 LifeCycle
   1.  Eclipse workspace 아래의 .metadata 폴더에서 \.plugins\org.eclipse.wst.server.core\tmp0\work\Catalina\localhost\firstweb\org\apache\jsp에 sum10_jsp.java 파일이 생성됨
   2.  해당 파일의 _jspService() 안을 살펴보면 jsp 파일의 내용이 변환되어 들어있는 것을 확인할 수 있다.
   3.  sum10_jsp.java는 서블릿 소스로 자동 컴파일 되면서 실행되어 결과가 Browser 상에 나타난다.



3. JSP 실행순서
   1. Browser가 WebServer에 JSP 요청을 전달
   2. Browser가 해당 JSP를 최초로 요청한 경우
      1. JSP로 작성된 코드를 Servlet 코드로 변환 (java 파일 생성)
      2. Servlet 코드를 컴파일해서 실행가능한 bytecode로 변환 (class 파일 생성)
      3. Servlet class를 로딩하고 instance를 생성
   3. Servlet이 실행되어 Request를 처리하고 Response 생성

ex) lifecycle.jsp
   ```
   <%@ page language="java" contentType="text/html; charset=UTF-8"
   	pageEncoding="UTF-8" %>
   <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
   <html>
       <head>
       <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
       <title>Insert title here</title>
   	</head>
       <body>
           <%
        		System.out.println("jspService()");
           %>

           <%! //jspService method의 바깥쪽에 코드를 넣을 수 있도록 해줌
           public void jspInit() {
            	   System.out.println("jspInit()");
           }
           public void jspDestroy() {
            	   System.out.println("jspDestroy()");
           }
           %>
       </body>
   </html>
   ```

4. JSP 문법
  - Declaration - <%! %> JSP 페이지 내에 필요한 전역변수 선언 및 메소드 선언
  - Scriptlet - <% %> 프로그래밍적 로직을 기술할 때 사용, 여기서 선언된 variable은 local variable
  - Expression - <%= %> 화면에 출력할 내용

ex1) Declaration
```Java
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
id: <%= getId() %>
<%!
	String id = "u001"; //global variable(멤버 변수?) 선언
	public String getId() { //method 선언
		return id;
	}
%>
</body>
</html>
```

ex2) Scriptlet
```Java
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
	for (int i = 1; i <=5; i++) {
%>		
<h<%=i %>>아름다운 한글</h<%=i %>>
<%
	}
%>

</body>
</html>
```

5. JSP 내부의 주석
  - Java의 주석 servlet에서 나타나고 html상에서 사라짐
  - JSP의 주석 servlet으로 변환하는 과정에서 사라짐
  - HTML의 주석 - HTML 문서상엔 존재하지만 브라우저에 나타나지 않음

ex)
```Java
// Java comment
<!-- HTML comment -->
<%-- JSP comment --%>
```


6. JSP의 내장객체
	- Servlet으로 변환되면서 선언됨. 그냥 가져다 쓰면 된다
	- ex) request, response, pageContext, session, application, out, config, page, exception 등등이 있다.

ex)
```Java
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
    StringBuffer url = request.getRequestURL();

    out.println("url : " + url.toString());
    out.println("<br>");
%>
</body>
</html>
```

7. Redirect
  - http protocol로 정해진 규칙
  - Client에서 request를 받은 후 특정 URL로 이동하라고 response를 보내는 것
  - Server에서 Client에게 Status code 302와 함께 이동할 URL정보를 Location Header에 담아 전송. Client는 Response가 Status code 302로 올 경우 Location Header값으로 다시 Request를 보낸다.
  - Servlet과 JSP에서는 HttpServletResponse의 sendRedirect()를 이용한다.

ex) redirect01.jsp
```Java
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%
	response.sendRedirect("redirect02.jsp");
%>
```

ex) redirect02.jsp
```Java
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
You are redirected to here
</body>
</html>
```

8. Expression language
  - 디자이너, FE 개발자와의 협업을 위해 Java코드를 줄이고 이해하기 쉬운 표현 방법을 찾기 위해 고안된 언어
  - 특징
      - 값을 표현하는데 사용되는 스크립트 언어
      - JSP의 기본 문법을 보완
      - JSP scope에 맞는 속성 사용
      - 집합 객체에 대한 접근 방법 제공
      - 수치 연산, 관계 연산, 논리 연산자 제공
      - 자바 클래스 메소드 호출 기능 제공
      - 표현언어만의 기본 객체 제공
  - Expression Language만의 표현 방법
  - JSP의 script요소(Declaration, Scriptlet, Expression)을 제외한 부분에서 사용 가능

ex)
기본 문법
```
Syntax
${expr} // expr - 표현 언어가 정의한 문법에 따라 값을 표현하는 식

Example
<jsp:include page="/module/${skin.id}/header.jsp" flush="true" />
<b>${sessionScope.member.id}</b>님 환영합니다.
```

ex2)
```
<%@ page contentType="text/html; charset=utf-8" %>
<%
  request.setAttribute("name", "홍길동");
%>
<html>
<head><title>EL Object</title></head>
<body>

요청 URI: ${pageContext.request.requestURI} <br/> <!-- pageContext.getRequest().getRequestURI() -->
request의 name 속성: ${requestScope.name} <br/> <!-- request.getAttribute("name") -->
code의 parameter ${param.code} <!-- request.getParameter("code") -->

</body>
</html>
```

  - Object 접근 규칙
      - expression1이나 expression2가 null인 경우 => null
      - expression1이 Map인 경우 expression2는 key로 처리됨
      - expression1이 List 또는 Array이고
          - expression2가 정수면 해당 expression2를 index로 처리
          - 정수가 아닌 경우 Error 발생
      - expression1이 객체인 경우 expression2에 해당하는 필드의 getter를 불러옴
```
    ${<expression1>.<expression2>}
```
  - 수치 연산자
      - +, -, *, / 또는 div, % 또는 mod
      - 숫자로 변환할 수 있는 경우 숫자 값으로 변환 후 수행
        - ${"10" + 1} <=> ${10 + 1}
      - 숫자로 변환할 수 없는 경우
        - ${"ten" + 1} => Error
      - null이면 0처리
        - ${null + 1} <=> ${0 + 1}

  - 비교 연산자
      - == 또는 eq, != 또는 ne, < 또는 lt, > 또는 gt, <= 또는 le, >= 또는 ge
      - HTML 문법과 겹치는 경우가 있으니 gt, lt 같은 문자로 사용하는게 좋다
      - String의 경우, ${str == "value"} <=> str.compareTo("value") == 0 <=> str.equals("value")

  - 논리 연산자
      - && 또는 Random
      - || 또는 or
      - != 또는 not
  - empty, ternary
      - empty <value>
          - null인 경우 true
          - ""인 경우 true
          - 비어있는 array, collection, map인 경우 => true
          - 이 외의 경우 false
      - <expression> ? <value1> : <value2>
          - true인 경우 value1
          - false인 경우 value2

  - 연산 우선순위
      1. [] .
      2. ()
      3. - (단일) not ! empty
      4. * / div % mod
      5. +-
      6. <> >= <= lt gt le ge
      7. == != eq ne
      8. && and
      9. || or
      10. ? :

  - Deactivate Expression Language
      - JSP에 아래의 코드를 넣으면 됨
```
    <%@ page is ELIgnored = "true" %> // 기본 값은 false
```

ex1) el01.jsp
```html
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<%
	pageContext.setAttribute("p1", "page scope value");
	request.setAttribute("r1", "request scope value");
	session.setAttribute("s1", "session scope value");
	application.setAttribute("a1", "application scope value");
%>

<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
<h2>EL을 이용한 값 가져오기</h2>

pageContext.getAttribute("p1"): <%=pageContext.getAttribute("p1") %> <br/>
pageContext.getAttribute("p1"): ${pageScope.p1} <br/>
request.getAttribute("r1"): ${requestScope.r1} <br/>
session.getAttribute("s1"): ${sessionScope.s1} <br/>
application.getAttribute("a1"): ${applicationScope.a1}<br/>


<h2>Key 값이 겹치지 않는 경우 XXXScope를 생략해도 값을 얻어올 수 있다.</h2>
pageContext.getAttribute("p1"): ${p1} <br/>
request.getAttribute("r1"): ${r1} <br/>
session.getAttribute("s1"): ${r1} <br/>
application.getAttribute("a1"): ${a1}<br/>

</body>
</html>
```

ex2) el02.jsp
```html
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ page isELIgnored="true" %>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<%
request.setAttribute("k", 10);
request.setAttribute("m", true);
%>

<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
<h1>EL을 이용한 사칙연산</h1>
k: ${k}<br/>
k + 5 : ${k + 5}<br/>
k - 5 : ${k - 5}<br/>
k * 5 : ${k * 5}<br/>
k / 5 : ${k div 5}<br/>

<h1>EL을 이용한 논리연산</h1>
k: ${k}<br/>
m: ${m}<br/>

k > 5 : ${ k > 5}<br/>
k < 5 : ${k < 5}<br/>
k <= 10 : ${k <= 10}<br/>
k >= 10 : ${k >= 10}<br/>
!m : ${!m}<br/>
</body>
</html>
```

9. JSTL(Java Standard Tag Library)
  - JSP페이지에서 조건문 처리, 반복문 처리 등을 html tag의 형태로 작성할 수 있도록 도와줌

ex) Scriptlet를 JSTL로 변환
```
<%
if (list.size() > 0) {
  for (int i = 0; i < list.size(); i++) {
    Data data = (Data) list.get(i);
%>
  <%= data.getTitle() %>
  ...

<%  
  }
} else {
%>
  No data found.
<%
}
%>
```
<=>
```
  <c:if test="!empty ${list}">
    <c:foreach varName="data" list="${list}">
      ${data.title}
    </c:foreach>
  </c:if>
  <c:if test="empty ${list}">
    No data found
  </c>
```
  - 사용 방법
      - http://tomcat.apache.org/download-taglibs.cgi 에 들어가서 아래의 jar 파일들을 다운로드 후 WEB-INF/lib폴더로 복사
          - taglibs-standard-impl-1.2.5.jar
          - taglibs-standard-spec-1.2.5.jar
          - taglibs-standard-jstlel-1.2.5.jar
  - JSTL에서 제공하는 tag 종류

| Library  | 하위 기능                            | Prefix | 관련 URI                               |
| -------- | ------------------------------------ | ------ | -------------------------------------- |
| core     | 변수지원, 흐름제어, URL처리          | c      | http://java.sun.com/jsp/jstl/core      |
| XML      | XML core, 흐름 제어, XML 변환        | x      | http://java.sun.com/jsp/jstl/xml       |
| Format   | 지역, 메세지 형식, 숫자 및 날짜 형식 | fmt    | http://java.sun.com/jsp/jstl/fmt       |
| Database | SQL                                  | sql    | http://java.sun.com/jsp/jstl/sql       |
| Function | Collection 처리, String 처리         | fn     | http://java.sun.com/jsp/jstl/functions |
  - Core tag 종류

|기능 분류|tag|설명|
|--------|---|----|
|변수 지원|set|JSP에서 사용될 변수 설정|
|        |remove|설정한 변수 제거|
|흐름 제어|if|조건에 따라 내부 코드를 수행|
|        |choose|다중 조건을 처리할 때 사용|
|        |forEach|Collection이나 Map의 각 항목을 처리할 때 사용|
|        |forTokens|구분자로 분리된 각각의 토큰을 처리할 때 사용|
|URL 처리|import|URL을 사용하여 다른 자원의 결과를 삽입|
|        |redirect|지정한 경로로 redirect|
|        |url|URL을 재작성|
|기타 tag들|catch|Exception Handling|
|         |out|JspWriter에 내용을 알맞게 처리한 후 출력|
  - 변수 지원 tag(set, remove)  사용 방법

set
```
<c:set var="varName" scope="session" value="someValue" />

<c:set var="varName" scope="request">
some Value
</c:set>
```

remove
```
<c:remove var="varName" scope="request" />
```

ex) jstl01.jsp
```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<c:set var="value1" scope="request" value="Lee" />
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
<h1>Printing name with JSTL</h1>
Last name: ${value1}<br/>
<c:remove var="value1" scope="request"/>
<h3>value1 is removed</h3>
Last name: ${value1}<br/>
</body>
</html>
```
  - 변수 지원 tag로 Property, Map 처리

```
<c:set target="${some}" property="propertyName" value="anyValue" />

some 객체가 JavaBean일 경우: some.setPropertyName(anyvalue);
some 객체가 JavaBean일 경우: some.put(propertyName, anyvalue);
```
  - Core tag의 흐름 제어 tag - if

```
<c:if test="condition">
...
...
</c:if>
```

ex) jstl02.jsp
```
<%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="UTF-8"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<c:set var="n" scope="request" value="10" />
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
	<c:if test="${n == 0 }">
		n is 0
	</c:if>
	<c:if test="${n == 10 }">
		n is 10
	</c:if>
</body>
</html>
```
  - Core tag의 흐름 제어 tag - choose
```
<c:choose>
  <c:when test="condition1"> condition1이 true면 실행
    ...
  </c:when>
  <c:when test="condition2">  condition2이 true면 실행
    ...
  </c:when>
  <c:otherwise> 앞의 모든 when절이 false인 경우 이 true면 실행
    ...
  </c:otherwise>
</c:choose>
```

ex)jstl03.jsp
```
<%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="UTF-8"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<% request.setAttribute("score", 83); %>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
	<c:choose>
		<c:when test="${score >= 90}">
			A
		</c:when>
		<c:when test="${score >= 80}">
			B
		</c:when>
		<c:when test="${score >= 70}">
			C
		</c:when>
		<c:when test="${score >= 60}">
			D
		</c:when>
		<c:otherwise>
			F
		</c:otherwise>
	</c:choose>
</body>
</html>
```

  - Core tag의 흐름 제어 tag - forEach
    - Array 및 Collection에 저장된 요소를 처리한다.

```
List로 받은 경우
<c:forEach var="thing" items="things" [begin="indexToBegin"] [end="indexToEnd"]>
...
${thing}
...
</c:forEach>

Map으로 받은 경우
<c:forEach var="thing" items="things" [begin="indexToBegin"] [end="indexToEnd"]>
...
key: ${thing.key}
value: ${thing.value}
...
</c:forEach>
```

ex)jstl04.jsp
```
<%@page import="java.util.List"%>
<%@page import="java.util.ArrayList"%>
<%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="UTF-8"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<%
	List<String> list = new ArrayList();

	list.add("Hello!");
	list.add("world");
	list.add("!!!!");

	request.setAttribute("list", list);
%>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
<h1>Printing using forEach</h1>
<c:forEach items="${list}" var="item" begin="1">
	${item } <br/>
</c:forEach>
</body>
</html>
```

  - Core tag 흐름제어 tag - import
```
<c:import url="URL" charEncoding="encoding" var="varName" scope="scope">
  <c:param name="parameterName" value="parameterValue" /> ?뒤에 넣어줄 URI parameter 지정
</c:import>
```

ex) jstl05.jsp
```
<%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="UTF-8"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<c:import url="http://localhost:8080/firstweb/jstlValue.jsp" var="urlValue" scope="request" />
<c:import url="http://www.google.com" var="urlValue2" scope="request" />
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
${urlValue}
${urlValue2}
</body>
</html>
```
ex) jstlValue.jsp
```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
John Doe
```
  - Core tag의 흐름 제어 tag - redirect
    - 지정한 page로 redirect처리
    - response.sendRedirect와 비슷

```
<c:redirect url="URL to redirect">
  <c:param name="paramName" value="paramValue" />  ?뒤에 넣어줄 URI parameter 지정
</c:redirect>
```

ex) jstl06.jsp
```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<c:redirect url="http://localhost:8080/firstweb/jstl05.jsp"></c:redirect>
```

  - Core tag의 기타 tag - out
    - JspWriting에 Data를 출력
    - value - JspWriter에 출력할 값을 나타낸다. 일반적으로 value의 값은 문자열이지만 java.io.Reader의 한 종류라면 out 태그는 Reader로 부터 data를 읽어와 JspWriter에 값을 출력
    - escapeXml - true인 경우 아래와 같이 문자를 변경. (생략가능, 생략시 기본값은 true)
      - < => `&lt;`
      - > => `&gt;`
      - & => `&amp;`
      - ' => `&#039;`
      - " => `&#034;`
    - default - value에 지정한 값이 없을 경우 사용되는 값

```
<c:out value="value" escapeXml="{true|false}" default="defaultValue" />
```

ex)jstl07.jsp
```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
<c:set var="t" value="<script type='text/javascript'>alert(1);</script>" />

<c:out value="${t}" escapeXml="false" />
<!--
escapeXml값이 true인 경우 출력값:
&lt;script type=&#039;text/javascript&#039;&gt;alert(1);&lt;/script&gt;
false인 경우 script가 실행됨
 -->

</body>
</html>
```

원본 강의: https://www.edwith.org/boostcourse-web
