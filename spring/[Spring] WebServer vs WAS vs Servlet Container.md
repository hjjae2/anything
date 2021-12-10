## WebServer vs WAS(Web Application Server) vs Servlet Container


### WebServer (웹 서버)

정적인 페이지(HTML, CSS)를 제공하는 서버이다.

클라이언트의 요청 중 자체적으로 처리할 수 없는 요청은 WAS로 넘긴다.

웹서버와 WAS, 즉 역할을 분리함으로써 요청에 대한 부담을 나눌 수 있다.

Apache, Nginx, IIS 등이 있다.

<br>

### WAS

서버 사이드 언어(코드)를 통해 동적인 콘텐츠를 만드는 서버이다.

웹 서버에서 처리할 수 없는 동적인 콘텐츠를 생성하고, 이것을 웹서버에 제공한다.

보통 웹서버의 기능을 포함하고 있어서 웹 서버 없이도 웹 서비스를 운영할 수 있다.

WAS 는 Servelet Container 개념을 포함한다.

Apache Tomcat, IBM WebSphere, Oracle WebLogic, Glassfish, JBoss 등이 있다.

**(WAS 내부의 Container들 중) Servlet Container의 실행 절차**

1. 웹서버로부터 요청이 들어오면, WAS 내의 Servlet Container 가 요청을 받는다.
2. Servlet Container 는 HttpServletRequest, HttpServletResponse 객체를 생성한다.
3. 배포서술자(web.xml)를 참조하여 Servlet 을 호출한다. (Thread를 생성한다.)
4. 호출된 Servlet은 service() 메소드를 호출하고, 요청에 맞게 doMethod() 메소드를 호출한다.
5. 호출된 doMethod() 메소드는 동적으로 페이지를 생성하고 HttpServletResponse 객체에 응답을 실어서 Servlet Container 에 전달한다. (아마도 이때 우리가 작성한 코드(Controller, Service, ...)가 동작하여 동적 페이지가 생성될 것이다.)
6. Container는 전달받은 Response 객체를 HTTPResponse 형태로 전환하여 웹서버에 전달한다. 생성되었던 Thread를 종료하고 HttpServletRequest, HttpServletResponse 객체를 소멸시킨다.

> (PHP의 경우는 WAS의 역할을 하지만 웹서버의 모듈 형태로 작동하므로 WAS라 불리는데 이견이 많다 ... ?)

<br>

### Servlet Container

WAS 별로 다양한 종류의 컨테이너를 내장하고 있다. 이들 중 Servlet 에 관련된 기능을 하는 것(모아둔 것)을 **Servlet Container** 라고 한다. (이 외에도 Servlet Container, JSP Container, EJB Container 등의 다양한 컨테이너가 있다.)

즉 서블릿을 관리해주는 컨테이너이다.

**주된 목적**
1. 생명 주기 관리<br>
   Servlet을 초기화, 호출, 소멸시킨다.
2. 통신 지원<br>
   웹서버로부터 받은 요청을 분석하여 Servlet을 실행시킨다.
3. 멀티쓰레딩 지원 (Multi-Threading)<br>
   클라이언트의 요청에 따라 Servlet을 생성하고, 이미 생성된 Servlet에 대한 요청은 스레드를 생성해 실행한다.
4. 보안 관리
5. JSP 지원

Apache Tomcat, 제티 등이 있다.


<br>

> Reference
> 1. https://pjh3749.tistory.com/267