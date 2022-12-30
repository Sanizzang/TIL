## 서블릿

---

서블릿은 톰캣 같은 웹 애플리케이션 서버를 직접 설치하고, 그 위에 서블릿 코드를 클래스 파일로 빌드해서 올린 다음, 톰캣 서버를 실행하면 된다. 하지만 이 과정은 매우 번거롭다.
스프맅 부트는 톰캣 서버를 내장하고 있으므로, 톰캣 서버 설치 없이 편리하게 서블릿 코드를 실행할 수 있다

#### @WebServlet(name=xxx, urlPatterns: xxx) 서블릿 애노테이션

- name: 서블릿 이름
- urlPatterns: URL 매핑

#### HTTP 요청 메시지 로그로 확인하기

application.properties

```properties
logging.level.org.apache.coyote.http11=debug
```

운영 서버에 이렇게 모든 요청 정보를 다 남기면 성능저하가 발생 할 수 있기 때문에, 개발 단계에서만 적용하자.

### 서블릿 컨테이너 동작 방식

1. 내장 톰캣 서버 생성
2. HTTP 요청, HTTP 응답 메시지
3. 웹 애플리케이션 서버의 요청 응답 구조

HTTP 응답에서 Content-Length는 웹 애플리케이션 서버가 자동으로 생성해준다.

#### welcome 페이지 추가

webapp 경로에 index.html을 두면 http://localhost:8080 호출시 index.html 페이지가 열린다.

### HttpServletRequest

#### HttpServletRequest 역할

HTTP 요청 메시지를 개발자가 직접 파싱해서 사용해도 되지만, 매우 불편할 것이다. 서블릿은 개발자가 HTTP 요청 메시지를 편리하게 사용할 수 있도록 개발자 대신에 HTTP 요청 메시지를 파싱한다. 그리고 그 결과를 HttpServletRequest 객체에 담아서 제공한다.

- HTTP 요청 메시지

```
POST /save HTTP/1.1
Host: localhost:8080
Content-Type: application/x-www-form-urlencoded

username=kim&age=20
```

- START LINE
  - HTTP 메소드
  - URL
  - 쿼리 스트링
  - 스키마, 프로토콜
- 헤더
  - 헤더 조회
- 바디
  - form 파라미터 형식 조회
  - message body 데이터 직접 조회

#### HttpServletRequest 객체의 부가기능

- 임시 저장소 기능

  - 해당 HTTP 요청이 시작부터 끝날 때까지 유지되는 임시 저장소 기능
    - 저장: request.setAttribute(name, value)
    - 조회: request.getAttribute(name)

- 세션 관리 기능
  - request.getSession(create: true)

### HTTP 요청 데이터 - 개요

- GET - 쿼리 파라미터
  - /url?username=hello&age=20
  - 메시지 바디 없이, URL의 쿼리 파라미터에 데이터를 포함해서 전달
  - 예) 검색 필터, 페이징등에서 많이 사용하는 방식
- POST - HTML Form
  - content-type: application/x-www-form-urlencoded
  - 메시지 바디에 쿼리 파라미터 형식으로 전달 username=hello&age=20
  - 예) 회원 가입, 상품 주문, HTML Form 사용
- HTTP message body에 데이터를 직접 담아서 요청
  - HTTP API에서 주로 사용, JSON, XML, TEXT
- 데이터 형식은 주로 JSON 사용
  - POST, PUT, PATCH

#### HTTP 요청 데이터 - GET 쿼리 파라미터

- 쿼리 파라미터 조회 메서드

```java
String username = request.getParameter("username"); //단일 파라미터 조회
Enumeration<String> parameterNames = request.getParameterNames(); //파라미터 이름들
모두 조회
Map<String, String[]> parameterMap = request.getParameterMap(); //파라미터를 Map
으로 조회
String[] usernames = request.getParameterValues("username"); //복수 파라미터 조회
```

#### HTTP 요청 데이터 - POST HTML Form

- 특징
  - content-type: application/x-www-form-urlencoded
  - 메시지 바디에 쿼리 파라미터 형식으로 데이터를 전달한다.

application/x-www-form-urlencoded 형식은 앞서 GET에서 살펴본 쿼리 파라미터 형식과 같다. 따라서 쿼리 파라미터 조회 메서드를 그대로 사용하면 된다. 클라이언트(웹 브라우저) 입장에서는 두 방식에 차이가 있지만, 서버 입장에서는 둘의 형식이 동일하므로, request.getParameter()로 편리하게 구분없이 조회할 수 있다.

절리하면 request.getParameter()는 GET URL 쿼리 파라미터 형식도 지원하고, POST HTML Form 형식도 둘 다 지원한다.

- 참고
  content-type은 HTTP 메시지 바디의 데이터 형식을 지정한다.
  GET URL 쿼리 파라미터 형식으로 클라이언트에서 서버로 데이터를 전달할 때는 HTTP 메시지 바디를
  사용하지 않기 때문에 content-type이 없다.
  POST HTML Form 형식으로 데이터를 전달하면 HTTP 메시지 바디에 해당 데이터를 포함해서 보내기
  때문에 바디에 포함된 데이터가 어떤 형식인지 content-type을 꼭 지정해야 한다. 이렇게 폼으로 데이터를
  전송하는 형식을 application/x-www-form-urlencoded 라 한다.

#### HTTP 요청 데이터 - API 메시지 바디 - 단순 텍스트

- HTTP message body에 데이터를 직접 담아서 요청
  - HTTP API에서 주로 사용, JSON, XML, TEXT
  - 데이터 형식은 주로 JSON 사용
  - POST, PUT, PATCH

```java
@WebServlet(name = "requestBodyStringServlet", urlPatterns = "/request-bodystring")
public class RequestBodyStringServlet extends HttpServlet {
 @Override
 protected void service(HttpServletRequest request, HttpServletResponse
response)
 throws ServletException, IOException {

 ServletInputStream inputStream = request.getInputStream();
 String messageBody = StreamUtils.copyToString(inputStream,
StandardCharsets.UTF_8);
 System.out.println("messageBody = " + messageBody);
 response.getWriter().write("ok");
 }
}
```

inputStream은 byte 코드를 반환한다. byte 코드를 우리가 읽을 수 있는 문자(String)로 보려면 문자표(Charset)를 지정해주어야 한다. 여기서는 UTF-8 Charset을 지정해주었다.

#### HTTP 요청 데이터 - API 메시지 바디 - JSON

- JSON 형식 전송

  - POST http://localhost:8080/request-body-json
  - content-type: application/json
  - message body: {"username": "hello", "age": 20}
  - 결과: messageBody = {"username": "hello", "age": 20}

- JSON 형식 파싱 추가
  JSON 형식으로 파싱할 수 있게 객체를 하나 생성

```java
@Getter @Setter
public class HelloData {
 private String username;
 private int age;
}
```

```java
@WebServlet(name = "requestBodyJsonServlet", urlPatterns = "/request-bodyjson")
public class RequestBodyJsonServlet extends HttpServlet {
 private ObjectMapper objectMapper = new ObjectMapper();
 @Override
 protected void service(HttpServletRequest request, HttpServletResponse
response)
 throws ServletException, IOException {
 ServletInputStream inputStream = request.getInputStream();
 String messageBody = StreamUtils.copyToString(inputStream,
StandardCharsets.UTF_8);
 System.out.println("messageBody = " + messageBody);
 HelloData helloData = objectMapper.readValue(messageBody,
HelloData.class);
 System.out.println("helloData.username = " + helloData.getUsername());
 System.out.println("helloData.age = " + helloData.getAge());
 response.getWriter().write("ok");
 }
}
```

- 참고
  JSON 결과를 파싱해서 사용할 수 있는 자바 객체로 변환하려면 Jackson, Gson 같은 JSON 변환 라이브러리를 추가해서 사용해야 한다. 스프링 부트로 Spring MVC를 선택하면 기본으로 Jackson 라이브러리(ObjectMapper)를 함께 제공한다.

- 참고
  HTML form 데이터도 메시지 바디를 통해 전송되므로 직접 읽을 수 있다. 하지만 편리한 파리미터 조회
  기능( request.getParameter(...) )을 이미 제공하기 때문에 파라미터 조회 기능을 사용하면 된다

### HttpServletResponse - 기본 사용법

#### HttpServletResponse 역할

- HTTP 응답 메시지 생성
  - HTTP 응답코드 지정
  - 헤더 생성
  - 바디 생성

#### HTTP 응답 데이터 - 단순 텍스트, HTML

- 단순 텍스트 응답 -앞에서 살펴봄 ( writer.println("ok"); )
- HTML 응답

```java
@WebServlet(name = "responseHtmlServlet", urlPatterns = "/response-html")
public class ResponseHtmlServlet extends HttpServlet {
 @Override
 protected void service(HttpServletRequest request, HttpServletResponse
response)
 throws ServletException, IOException {
 //Content-Type: text/html;charset=utf-8
 response.setContentType("text/html");
 response.setCharacterEncoding("utf-8");
 PrintWriter writer = response.getWriter();
 writer.println("<html>");
 writer.println("<body>");
 writer.println(" <div>안녕?</div>");
 writer.println("</body>");
 writer.println("</html>");
 }
}
```

HTTP 응답으로 HTML을 반환할 떄는 content-type을 text/html로 지정해야 한다.

- HTTP API - MessageBody JSON 응답

```java
@WebServlet(name = "responseJsonServlet", urlPatterns = "/response-json")
public class ResponseJsonServlet extends HttpServlet {
 private ObjectMapper objectMapper = new ObjectMapper();
 @Override
 protected void service(HttpServletRequest request, HttpServletResponse
response)
 throws ServletException, IOException {
 //Content-Type: application/json
 response.setHeader("content-type", "application/json");
 response.setCharacterEncoding("utf-8");
 HelloData data = new HelloData();
 data.setUsername("kim");
 data.setAge(20);
 //{"username":"kim","age":20}
 String result = objectMapper.writeValueAsString(data);
 response.getWriter().write(result);
 }
}
```

HTTP 응답으로 JSON을 반환할 때는 content-type을 application/json 로 지정해야 한다.
Jackson 라이브러리가 제공하는 objectMapper.writeValueAsString() 를 사용하면 객체를 JSON
문자로 변경할 수 있다.

## 서블릿, JSP, MVC 패턴

---

### MVC 패턴의 등장

비즈니스 로직은 서블릿 처럼 다른곳에서 처리하고, JSP는 목적에 맞게 HTML로 화면(View)을 그리는
일에 집중한다.

- 컨트롤러: HTTP 요청을 받아서 파라미터를 검증하고, 비즈니스 로직을 실행한다. 그리고 뷰에 전달할 결과 데이터를 조회해서 모델에 담는다.

- 모델: 뷰에 출력할 데이터를 담아둔다. 뷰가 필요한 데이터를 모두 모델에 담아서 전달해주는 덕분에 뷰는 비즈니스 로직이나 데이터 접근을 몰라도 되고, 화면을 렌더링 하는 일에 집중할 수 있다.

- 뷰: 모델에 담겨있는 데이터를 사용해서 화면을 그리는 일에 집중한다. 여기서는 HTML을 생성하는 부분을
  말한다.

컨트롤러에 비즈니스 로직을 둘 수도 있지만, 이렇게 되면 컨트롤러가 너무 많은 역할을 담당한다. 그래서
일반적으로 비즈니스 로직은 서비스(Service)라는 계층을 별도로 만들어서 처리한다. 그리고 컨트롤러는
비즈니스 로직이 있는 서비스를 호출하는 역할을 담당한다. 참고로 비즈니스 로직을 변경하면 비즈니스
로직을 호출하는 컨트롤러의 코드도 변경될 수 있다.

### MVC 패턴 - 적용

서블릿을 컨트롤러로 사용하고 , JSP를 뷰로 사용해서 MVC 패턴을 적용
Model은 HttpServletRequest 객체를 사용한다. request는 내부에 데이터 저장소를 가지고 있는데, request.setAttribute(), request.getAttribute()를 사용하면 데이터를 보관하고, 조회할 수 있다.

- 회원 등록 폼 - 컨트롤러

```java
@WebServlet(name = "mvcMemberFormServlet", urlPatterns = "/servlet-mvc/members/
new-form")
public class MvcMemberFormServlet extends HttpServlet {
 @Override
 protected void service(HttpServletRequest request, HttpServletResponse
response)
 throws ServletException, IOException {
 String viewPath = "/WEB-INF/views/new-form.jsp";
 RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
 dispatcher.forward(request, response);
 }
}
```

dispatcher.forward(): 다른 서블릿이나 JSP로 이동할 수 있는 기능이다. 서버 내부에서 다시 호출이 발생한다.

- /WEB-INF
  이 경로안에 JSP가 있으면 외부에서 직접 JSP를 호출할 수 없다. 우리가 기대하는 것은 항상 컨트롤러를 통해서 JSP를 호출하는 것이다.

- redirect vs forward
  리다이렉트는 실제 클라이언트(웹 브라우저)에 응답이 나갔다가, 클라이언트가 redirect 경로로 다시 요청한다. 따라서 클라이언트가 인지할 수 있고, URL 경로도 실제로 변경된다. 반면에 포워드는 서버 내부에서 일어나는 호출이기 떄문에 클라이언트가 전혀 인지하지 못한다.

- 회원 등록 폼 - 뷰

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
 <meta charset="UTF-8">
 <title>Title</title>
</head>
<body>
<!-- 상대경로 사용, [현재 URL이 속한 계층 경로 + /save] -->
<form action="save" method="post">
 username: <input type="text" name="username" />
 age: <input type="text" name="age" />
 <button type="submit">전송</button>
</form>
</body>
</html>
```

### MVC 패턴 - 한계

MVC 패턴을 적용한 덕분에 컨트롤러의 역할과 뷰를 렌더링 하는 역할을 명확하게 구분할 수 있다.
특히 뷰는 화면을 그리는 역할에 충실한 덕분에, 코드가 깔끔하고 직관적이다. 단순하게 모델에서 필요한
데이터를 꺼내고, 화면을 만들면 된다.
그런데 컨트롤러는 딱 봐도 중복이 많고, 필요하지 않는 코드들도 많이 보인다.

- 포워드 중복
  View로 이동하는 코드가 항상 중복 호출되어야 한다. 물론 이 부분을 메서드로 공통화해도 되지만, 해당
  메서드도 항상 직접 호출해야 한다.

```java
RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
dispatcher.forward(request, response);
```

- ViewPath에 중복

```java
String viewPath = "/WEB-INF/views/new-form.jsp";
```

- 사용하지 않는 코드

```java
HttpServletRequest request, HttpServletResponse response
```

- 공통 처리가 어렵다.
  기능이 복잡해질 수 록 컨트롤러에서 공통으로 처리해야 하는 부분이 점점 더 많이 증가할 것이다. 단순히
  공통 기능을 메서드로 뽑으면 될 것 같지만, 결과적으로 해당 메서드를 항상 호출해야 하고, 실수로 호출하지
  않으면 문제가 될 것이다. 그리고 호출하는 것 자체도 중복이다.

- 정리하면 공통 처리가 어렵다는 문제
  이 문제를 해결하려면 컨트롤러 호출 전에 먼저 공통 기능을 처리해야 한다. 소위 수문장 역할을 하는 기능이
  필요하다. 프론트 컨트롤러(Front Controller) 패턴을 도입하면 이런 문제를 깔끔하게 해결할 수 있다.
  (입구를 하나로!)
  스프링 MVC의 핵심도 바로 이 프론트 컨트롤러에 있다

## MVC 프레임워크 만들기

### FrontController 패턴 특징

- 프론트 컨트롤러 서블릿 하나로 클라이언트 요청을 받음
- 프론트 컨트롤러가 요청에 맞는 컨트롤러를 찾아서 호출
- 입구를 하나로!
- 공통 처리 가능
- 프론트 컨트롤러를 제외한 나머지 컨트롤러는 서블릿을 사용하지 않아도 됨

#### 스프링 웹 MVC와 프론트 컨트롤러

스프링 웹 MVC의 핵심도 바로 FrontController
스프링 웹 MVC의 DispatcherServlet이 FrontController 패턴으로 구현되어 있음

## 스프링 MVC - 구조 이해

스프링 MVC도 프론트 컨트롤러 패턴으로 구현되어 있다. 스프링 MVC의 프론트 컨트롤러가 바로 디스패처 서블릿(Dispatcher Servlet)이다. 그리고 이 디스패처 서블릿이 바로 스프링 MVC의 핵심이다.

### DispatcherServlet 서블릿 등록

- DispatcherServlet도 부모 클래스에서 HttpServlet을 상속 받아서 사용하고, 서블릿으로 동작한다.
  - DispatcherServlet -> FrameworkServlet -> HttpServletBean -> HttpServlet
- 스프링 부트는 DispatcherServlet을 서블릿으로 자동으로 등록하면서 모든 경로(urlPatters="/")에 대해서 매핑한다.

#### 요청 흐름

- 서블릿이 호출되면 HttpServlet이 제공하는 service()가 호출된다.
- 스프링 MVC는 DispatcherServlet의 부모인 FrameworkServlet에서 service()를 오버라이드 해두었다.
- FrameworkServlet.service()를 시작으로 여러 메서드가 호출되면서 DispatcherServlet.doDispatch()가 호출된다.

#### SpringMVC 구조

- 동작 순서

1. 핸들러 조회: 핸들러 매핑을 통해 요청 URL에 매핑된 핸들러(컨트롤러)를 조회한다.
2. 핸들러 어댑터 조회: 핸들러를 실행할 수 있는 핸들러 어댑터를 조회한다.
3. 핸들러 어댑터 실행: 핸들러 어댑터를 실행한다.
4. 핸들러 실행: 핸들러 어댑터가 실제 핸들러를 실행한다.
5. ModelAndView 반환: 핸들러 어댑터는 핸들러가 반환하는 정보를 ModelAndView로 변환해서 반환한다.
6. viewResolver 호출: 뷰 리졸버를 찾고 실행한다.

- JSP의 경우: InternalResourceViewResolver가 자동 등록되고, 사용된다.

7. View 반환: 뷰 리졸버는 뷰의 논리 이름을 물리 이름으로 바꾸고, 렌더링 역할을 담당하는 뷰 객체를 반환한다.

- JSP의 경우 IntervalResourceView(JstlView)를 반환하는데, 내부에 forward() 로직이 있다.

8. 뷰 렌더링: 뷰를 통해서 뷰를 렌더링 한다.

### 스프링 부트가 자동 등록하는 핸들러 매핑과 핸들러 어댑터

#### HandlerMapping

```
0 = RequestMappingHandlerMapping : 애노테이션 기반의 컨트롤러인 @RequestMapping에서 사용
1 = BeanNameUrlHandlerMapping : 스프링 빈의 이름으로 핸들러를 찾는다.
```

#### HandlerAdapter

```
0 = RequestMappingHandlerAdapter : 애노테이션 기반의 컨트롤러인 @RequestMapping에서 사용
1 = HttpRequestHandlerAdapter : HttpRequestHandler 처리
2 = SimpleControllerHandlerAdapter : Controller 인터페이스 (애노테이션X, 과거에 사용)
```

핸들러 매핑도, 핸들러 어댑터도 모두 순서대로 찾고 만약 없으면 다음 순서로 넘어간다.

#### 뷰 리졸버 - InternalResourceViewResolver

스프링 부트는 InternalResourceViewResolver라는 뷰 리졸버를 자동으로 등록하는데, 이때 application.properties에 등록한 spring.mvc.view.prefix, spring.mvc.view.suffix 설정 정보를 사용해서 등록한다.

#### 스프링 부트가 자동 등록하는 뷰 리졸버

```
1 = BeanNameViewResolver : 빈 이름으로 뷰를 찾아서 반환한다. (예: 엑셀 파일 생성 기능에 사용)
2 = InternalResourceViewResolver : JSP를 처리할 수 있는 뷰를 반환
```

### 스프링 MVC - 시작하기

스프링이 제공하는 컨트롤러는 애노테이션 기반으로 동작해서, 매우 유연하고 실용적이다. 과거에는 자바언어에 애노테이션이 없기도 했고, 스프링도 처음부터 이런 유연한 컨트롤러를 제공한 것은 아니다.

#### @RequestMapping

스프링은 애노테이션을 활용한 매우 유연하고, 실용적인 컨트롤러를 만들었는데 이것이 바로
@RequestMapping 애노테이션을 사용하는 컨트롤러이다. 다들 한번쯤 사용해보았을 것이다.
여담이지만 과거에는 스프링 프레임워크가 MVC 부분이 약해서 스프링을 사용하더라도 MVC 웹 기술은
스트럿츠 같은 다른 프레임워크를 사용했었다. 그런데 @RequestMapping 기반의 애노테이션 컨트롤러가
등장하면서, MVC 부분도 스프링의 완승으로 끝이 났다.

```java
@Controller
public class SpringMemberFormControllerV1 {
 @RequestMapping("/springmvc/v1/members/new-form")
 public ModelAndView process() {
 return new ModelAndView("new-form");
 }
}
```

- @Controller
  - 스프링이 자동으로 스프링 빈으로 등록한다. (내부에 @Component 애노테이션이 있어서 컴포넌트 스캔의 대상이 됨)
  - 스프링 MVC에서 애노테이션 기반 컨트롤러로 인식한다.
- @RequestMapping: 요청 정보를 매핑한다. 해당 URL이 호출되면 이 메서드가 호출된다. 애노테이션을 기반으로 동작하기 떄문에, 매서드의 이름은 임의로 지으면 된다.
- ModelAndView: 모델과 뷰 정보를 담아서 반환하면 된다.

RequestMapppingHandlerMapping은 스프링 빈 중에서 @RequestMapping 또는 @Controller가 클래스 레벨에 붙어 있는 경우에 매핑 정보로 인식한다.

```java
@Component //컴포넌트 스캔을 통해 스프링 빈으로 등록
@RequestMapping
public class SpringMemberFormControllerV1 {
 @RequestMapping("/springmvc/v1/members/new-form")
 public ModelAndView process() {
 return new ModelAndView("new-form");
 }
}
```

```java
@Controller
public class SpringMemberSaveControllerV1 {
 private MemberRepository memberRepository = MemberRepository.getInstance();
 @RequestMapping("/springmvc/v1/members/save")
 public ModelAndView process(HttpServletRequest request, HttpServletResponse
response) {
 String username = request.getParameter("username");
 int age = Integer.parseInt(request.getParameter("age"));
 Member member = new Member(username, age);
 System.out.println("member = " + member);
 memberRepository.save(member);
 ModelAndView mv = new ModelAndView("save-result");
 mv.addObject("member", member);
 return mv;
 }
}
```

- mv.addObject("member", member)
  - 스프링이 제공하는 ModelAndView를 통해 Model 데이터를 추가할 때는 addObject()를 사용하면 된다. 이 데이터는 이후 뷰를 렌더링 할 때 사용된다.

### 스프링 MVC - 실용적인 방식

```java
@Controller
@RequestMapping("/springmvc/v3/members")
public class SpringMemberControllerV3 {
 private MemberRepository memberRepository = MemberRepository.getInstance();
 @GetMapping("/new-form")
 public String newForm() {
 return "new-form";
 }
 @PostMapping("/save")
 public String save(
 @RequestParam("username") String username,
 @RequestParam("age") int age,
 Model model) {
 Member member = new Member(username, age);
 memberRepository.save(member);
 model.addAttribute("member", member);
 return "save-result";
 }
 @GetMapping
 public String members(Model model) {
 List<Member> members = memberRepository.findAll();
 model.addAttribute("members", members);
 return "members";
 }
}
```

- Model 파라미터

save(), members()를 보면 Model을 파라미터로 받는 것을 확인할 수 있다. 스프링 MVC도 이런 편의 기능을 제공한다.

- ViewName 직접 반환
  뷰의 논리 이름을 반환할 수 있다.

- @RequestParam 사용
  스프링은 HTTP 요청 파라미터를 @RequestParam으로 받을 수 있다. @RequestParam("username")은 request.getParameter("username")와 거의 같은 코드라 생각하면 된다. 물론 GET 쿼리 파라미터, POST Form 방식을 모두 지원한다.

- @RequestMapping -> @GetMapping, @PostMapping
  @RequestMapping 은 URL만 매칭하는 것이 아니라, HTTP Method도 함께 구분할 수 있다.
  예를 들어서 URL이 /new-form 이고, HTTP Method가 GET인 경우를 모두 만족하는 매핑을 하려면
  다음과 같이 처리하면 된다.

```java
@RequestMapping(value = "/new-form", method = RequestMethod.GET)
```

이것을 @GetMapping , @PostMapping 으로 더 편리하게 사용할 수 있다.
참고로 Get, Post, Put, Delete, Patch 모두 애노테이션이 준비되어 있다

## 스프링 MVC - 기본 기능

Jar를 사용하면 항상 내장 서버(톰캣등)를 사용하고, webapp 경로도 사용하지 않는다. 내장 서버 사용에 최적화 되어 있는 기능이다. 최근에는 주로 이 방식을 사용한다. War를 사용하면 내장 서버도 사용가능 하지만, 주로 외부 서버에 배포하는 목적으로 사용한다.

#### Welcome 페이지 만들기

스프링 부트에 Jar를 사용하면 /resources/static/ 위치에 index.html 파일을 두면 Welcome 페이지로 처리해준다. (스프링 부트가 지원하는 정적 컨텐츠 위치에 /index.html이 있으면 된다.)

### 로깅 간단히 알아보기

운영 시스템에서는 System.out.println()같은 시스템 콘솔을 사용해서 필요한 정보를 출력하지 않고, 별도의 로깅 라이브러리를 사용해서 로그를 출력한다. 참고로 로그 관련 라이브러리도 많고, 깊게 들어가면 끝이 없기 때문에, 여기서는 최소한의 사용 방법만 알아본다.

#### 로깅 라이브러리

스프링 부트 라이브러리를 사용하면 스프링 부트 로깅 라이브러리(spring-boot-starter-logging)가 함께 포함된다.
스프링 부트 로깅 라이브러리는 기본으로 다음 로깅 라이브러리를 사용한다.

- SLF4J - http://www.slf4j.org
- Logback - http://logback.qos.ch

로그 라이브러리는 Logback, Log4J, Log4J2 등등 수 많은 라이브러리가 있는데, 그것을 통합해서
인터페이스로 제공하는 것이 바로 SLF4J 라이브러리다.
쉽게 이야기해서 SLF4J는 인터페이스이고, 그 구현체로 Logback 같은 로그 라이브러리를 선택하면 된다.
실무에서는 스프링 부트가 기본으로 제공하는 Logback을 대부분 사용한다.

- 로그 선언

```java
private Logger log = LoggerFactory.getLogger(getClass());
```

#### 매핑 정보

- @RestController

  - @Controller는 반환 값이 String이면 뷰 이름으로 인식된다. 그래서 뷰를 찾고 뷰가 렌더링 된다.
  - @RestController는 반환 값으로 뷰를 찾는 것이 아니라, HTTP 메시지 바디에 바로 입력한다. 따라서 실행 결과로 ok메세지를 받을 수 있다.

- 로그 포맷
  - 시간, 로그 레벨, 프로세스 ID, 쓰레드 명, 클래스 명, 로그 메시지
- 로그 레벨
  - TRACE > DEBUG > INFO > WARN > ERROR
  - 개발 서버는 debug 출력
  - 운영 서버는 info 출력
- @Slf4j로 변경

#### 로그 레벨 설정

application.properties

```
#전체 로그 레벨 설정(기본 info)
logging.level.root=info

#hello.springmvc 패키지와 그 하위 로그 레벨 설정
logging.level.hello.springmvc=debug
```

- 올바를 로그 사용법

```java
log.debug("data={}", data)
```

- 로그 출력 레벨을 info로 설정하면 아무일도 발생하지 않는다.

#### 로그 사용시 장점

- 쓰레드 정보, 클래스 이름 같은 부가 정보를 함께 볼 수 있고, 출력 모양을 조정할 수 있다.
- 로그 레벨에 따라 개발 서버에서는 모든 로그를 출력하고, 운영서버에서는 출력하지 않는 등 로그를 상황에
  맞게 조절할 수 있다.
- 시스템 아웃 콘솔에만 출력하는 것이 아니라, 파일이나 네트워크 등, 로그를 별도의 위치에 남길 수 있다.
  특히 파일로 남길 때는 일별, 특정 용량에 따라 로그를 분할하는 것도 가능하다.
- 성능도 일반 System.out보다 좋다. (내부 버퍼링, 멀티 쓰레드 등등) 그래서 실무에서는 꼭 로그를
  사용해야 한다

### HTTP 메시지 컨버터

뷰 템플릿으로 HTML을 생성해서 응답하는 것이 아니라, HTTP API처럼 JSON 데이터를 HTTP 메시지 바디에서 직접 읽거나 쓰는 경우 HTTP 메시지 컨버터를 사용하면 편리하다.

#### 스프링 MVC는 다음의 경우에 HTTP 메시지 컨버터를 적용한다.

- HTTP 요청: @RequestBody, HttpEntity(RequestEntity),
- HTTP 응답: @ResponseBody, HttpEntity(ResponseEntity),

#### HTTP 메시지 컨버터 인터페이스

```java
public interface HttpMessageConverter<T> {
  boolean canRead(Class<?> clazz, @Nullable MediaType mediaType);
  boolean canWrite(Class<?> clazz, @Nullable MediaType mediaType);
  List<MediaType> getSupportedMediaTypes();
  T read(Class<? extends T> clazz, HttpInputMessage inputMessage)
  throws IOException, HttpMessageNotReadableException;
  void write(T t, @Nullable MediaType contentType, HttpOutputMessage
  outputMessage)
  throws IOException, HttpMessageNotWritableException;
}
```

HTTP 메시지 컨버터는 HTTP 요청, HTTP 응답 둘 다 사용된다.

- canRead(), canWrite(): 메시지 컨버터가 해당 클래스, 미디어타입을 지원하는지 체크
- read(), write(): 메시지 컨버터를 통해서 메시지를 읽고 쓰는 기능

#### 스프링 부트 기본 메시지 컨버터

```
0 = ByteArrayHttpMessageConverter
1 = String HttpMessageConverter
2 = MappingJackson2HttpMessageConverter
```

스프링 부트는 다양한 메시지 컨버터를 제공하는데, 대상 클래스 타입과 미디어 타입 둘을 체크해서 사용여부를 결정한다. 만약 만족하지 않으면 다음 메시지 컨버터로 우선순위가 넘어간다.

- ByteArrayHttpMessageConverter : byte[] 데이터를 처리한다.
  - 클래스 타입: byte[] , 미디어타입: _/_ ,
  - 요청 예) @RequestBody byte[] data
  - 응답 예) @ResponseBody return byte[] 쓰기 미디어타입 application/octet-stream
- StringHttpMessageConverter : String 문자로 데이터를 처리한다.
  - 클래스 타입: String , 미디어타입: _/_
  - 요청 예) @RequestBody String data
- 응답 예) @ResponseBody return "ok" 쓰기 미디어타입 text/plain
- MappingJackson2HttpMessageConverter : application/json
  - 클래스 타입: 객체 또는 HashMap , 미디어타입 application/json 관련
  - 요청 예) @RequestBody HelloData data
  - 응답 예) @ResponseBody return helloData 쓰기 미디어타입 application/json 관련

#### HTTP 요청 데이터 읽기

- HTTP 요청이 오고, 컨트롤러에서 @RequestBody, HttpEntity 파라미터를 사용한다.
- 메시지 컨버터가 메시지를 읽을 수 있는지 확인하기 위해 canRead()를 호출한다.
  - 대상 클래스 타입을 지원하는가.
    - 예) @RequestBody의 대상 클래스 (byte[], String, HelloData)
  - HTTP 요청의 Content-Type 미디어 타입을 지원하는가.
    - 예) text/plain, application/json, _/_
- canRead() 조건을 만족하면 read()를 호출해서 객체 생성하고, 반환한다.

#### HTTP 응답 데이터 생성

- 컨트롤러에서 @ResponseBody , HttpEntity 로 값이 반환된다.
- 메시지 컨버터가 메시지를 쓸 수 있는지 확인하기 위해 canWrite() 를 호출한다.
  - 대상 클래스 타입을 지원하는가.
    - 예) return의 대상 클래스 ( byte[] , String , HelloData )
  - HTTP 요청의 Accept 미디어 타입을 지원하는가.(더 정확히는 @RequestMapping 의 produces )
    - 예) text/plain , application/json , _/_
- canWrite() 조건을 만족하면 write() 를 호출해서 HTTP 응답 메시지 바디에 데이터를 생성한다.

#### ArgumentResolver

생각해보면, 애노테이션 기반의 컨트롤러는 매우 다양한 파라미터를 사용할 수 있었다.
HttpServletRequest , Model 은 물론이고, @RequestParam , @ModelAttribute 같은 애노테이션
그리고 @RequestBody , HttpEntity 같은 HTTP 메시지를 처리하는 부분까지 매우 큰 유연함을
보여주었다.
이렇게 파라미터를 유연하게 처리할 수 있는 이유가 바로 ArgumentResolver 덕분이다.
애노테이션 기반 컨트롤러를 처리하는 RequestMappingHandlerAdapter 는 바로 이
ArgumentResolver 를 호출해서 컨트롤러(핸들러)가 필요로 하는 다양한 파라미터의 값(객체)을 생성한다.
그리고 이렇게 파리미터의 값이 모두 준비되면 컨트롤러를 호출하면서 값을 넘겨준다.
스프링은 30개가 넘는 ArgumentResolver 를 기본으로 제공한다.

- 정확히는 HandlerMethodArgumentResolver 인데 줄여서 ArgumentResolver 라고 부른다.

```java
public interface HandlerMethodArgumentResolver {
  boolean supportsParameter(MethodParameter parameter);
  @Nullable
  Object resolveArgument(MethodParameter parameter, @Nullable
  ModelAndViewContainer mavContainer,
  NativeWebRequest webRequest, @Nullable WebDataBinderFactory
  binderFactory) throws Exception;
}
```

- 동작 방식
  ArgumentResolver 의 supportsParameter() 를 호출해서 해당 파라미터를 지원하는지 체크하고,
  지원하면 resolveArgument() 를 호출해서 실제 객체를 생성한다. 그리고 이렇게 생성된 객체가 컨트롤러
  호출시 넘어가는 것이다

#### RerturnValueHandler

HandlerMethodReturnValueHandler 를 줄여서 ReturnValueHandler 라 부른다.
ArgumentResolver 와 비슷한데, 이것은 응답 값을 변환하고 처리한다.
컨트롤러에서 String으로 뷰 이름을 반환해도, 동작하는 이유가 바로 ReturnValueHandler 덕분이다.
어떤 종류들이 있는지 살짝 코드로 확인만 해보자.
스프링은 10여개가 넘는 ReturnValueHandler 를 지원한다.
예) ModelAndView , @ResponseBody , HttpEntity , String

#### HTTP 메시지 컨버터 위치

HTTP 메시지 컨버터를 사용하는 @RequestBody 도 컨트롤러가 필요로 하는 파라미터의 값에 사용된다.
@ResponseBody 의 경우도 컨트롤러의 반환 값을 이용한다.
요청의 경우 @RequestBody 를 처리하는 ArgumentResolver 가 있고, HttpEntity 를 처리하는
ArgumentResolver 가 있다. 이 ArgumentResolver 들이 HTTP 메시지 컨버터를 사용해서 필요한
객체를 생성하는 것이다. (어떤 종류가 있는지 코드로 살짝 확인해보자)
응답의 경우 @ResponseBody 와 HttpEntity 를 처리하는 ReturnValueHandler 가 있다. 그리고
여기에서 HTTP 메시지 컨버터를 호출해서 응답 결과를 만든다.
스프링 MVC는 @RequestBody @ResponseBody 가 있으면
RequestResponseBodyMethodProcessor (ArgumentResolver)
HttpEntity 가 있으면 HttpEntityMethodProcessor (ArgumentResolver)를 사용한다.
