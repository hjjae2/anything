## Servlet

자바를 사용하여 웹을 만들기 위해 필요한 기술이다.

클라이언트의 요청을 처리하고, 결과를 반환하는 (Servlet 클래스의 구현 규칙을 지킨) 자바 프로그램(클래스)이다.

서블릿은 자바로 구현된 CGI라고 불린다.

> **\* 실제 주석 내용**<br>
> 서블릿은 웹 서버 내에서 실행되는 작은 Java 프로그램이다.
> 일반적으로 서블릿은 HTTP 를 통해 웹 클라이언트의 요청을 받고, 응답한다.

> **\* CGI(Common GateWay Interface)란?**<br>
외부 프로그램과 웹서버 사이에서 정보를 주고받는 방법/규약을 말한다.

**Servlet Interface**

이 인터페이스는 서블릿을 초기화하고 요청을 서비스하고, 서블릿을 서버로부터 삭제하는 메소드를 정의하고 있다. 즉, 생명 주기 메소드들을 정의한다.

1. 서블릿이 생성되고, init() 메소드로 초기화된다.
2. clients -> service 로의 모든 calls(호출) 이 처리된다.
3. 서블릿은 서비스에서 제외된다. 그 후 destroy() 메소드로 소멸된다. 그 후 GC에 의해 수집되고, 종료된다.

- init()
- service()
- destroy()
- getServiceInfo()<br>
  서블릿에게 기본 정보(author, version, copyright 등)를 리턴하도록 하는 메소드이다
- getServletConfig()<br>
  서블릿이 시작 정보를 get 할 수 있도록 사용하는 메소드이다.

**ServletConfig Interface**

서블릿을 초기화하는 동안, 서블릿 컨테이너가 서블릿에게 정보를 전달하기 위해 사용하는 Servlet configuration object 이다.

- getServletName()
- getServletContext()
- getInitParameter()
- getInitParameterNames()

**GenericServlet Abstract Class**

프로토콜에 독립적인(상관없는) generic 한 서블릿 클래스이다. (즉, 특정 프로토콜을 위해 사용되는 서블릿 클래스가 아니고 한단계 추상적인 의미로 사용될 수 있는 generic 한 서블릿 클래스이다.)

Servlet, ServletConfig 인터페이스를 구현했다. (Servlet 기능을 구현)

GenericServlet 을 상속받아 구현한 사용자 서블릿은, 사용되는 프로토콜에 따라 service()를 오버라이딩해서 구현한다.

**HttpServlet Abstract Class**

GenericServlet 을 상속받아 HTTP 프로토콜을 사용하는 웹 브라우저에서 서블릿 기능을 수행한다.

위의 GenericServlet 클래스는 말 그대로 generic 한 클래스이다. **웹 (프로토콜) 전용으로 사용하기 위한 Servlet 이 바로 HttpServlet 클래스이다. (웹 서비스를 제공하는 서블릿을 만들 때 사용한다.)**

(위의 것들을 구현, 상속하고 있으니) doMethod() 들과 service() 메소드가 있다.

이 중 service() 메소드는 오버라이딩할 이유가 거의 없다. (표준 HTTP 요청에 대해서) 각의각 HTTP 요청을 각각의 핸들러 메소드(doMethod()) 로 보내는 메소드이기 때문이다.

- 클라이언트의 요청 시, service() 가 호출되고 클라이언트의 요청 방식에 따라 doGet(), doPost(), doPut() 등의 메소드가 호출된다.

> 서블릿은 멀티쓰레드 환경에서 처리될 수 있는 클래스이다. 이것을 분석하면 멀티쓰레드 환경에서의 코드 기법을 알 수 있지 않을까 싶다.

<br>

>Reference
> 1. https://mangkyu.tistory.com/14