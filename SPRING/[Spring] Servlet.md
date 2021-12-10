## Servlet

**클라이언트의 요청을 처리하고, 결과를 반환하는 (Servlet 클래스의 구현 규칙을 지킨) Java 프로그램(클래스)이다.**

서블릿은 자바로 구현된 CGI라고 불린다.

> **\* 실제 주석 내용**<br>
> 서블릿은 웹 서버 내에서 실행되는 작은 Java 프로그램이다.
> 일반적으로 서블릿은 HTTP 를 통해 웹 클라이언트의 요청을 받고, 응답한다.

> **\* CGI(Common GateWay Interface)란?**<br>
외부 프로그램과 웹서버 사이에서 정보를 주고받는 방법/규약을 말한다.

<br><br>

### Servlet (Interface)

이 인터페이스는 서블릿을 초기화하고 요청을 서비스하고, 서블릿을 서버로부터 삭제하는 메소드를 정의하고 있다. <u>즉, 생명 주기 메소드들을 정의한다.</u>

```java
/**
A servlet is a small Java program that runs within a Web server. Servlets receive and respond to requests from Web clients, usually across HTTP, the HyperText Transfer Protocol.

To implement this interface, you can write a generic servlet that extends javax.servlet.GenericServlet or an HTTP servlet that extends javax.servlet.http.HttpServlet.

...
*/
public interface Servlet {

  /**
  Called by the servlet container to indicate 
  to a servlet that the servlet is being placed into service.
  */
  init(ServletConfig): void


  /**
  Called by the servlet container to allow the servlet to respond to a request. 
  This method is only called after the servlet's init() method has completed successfully.

  Servlets typically run inside multithreaded servlet containers that can handle multiple requests concurrently.
  Developers must be aware to synchronize access to any shared resources such as files, 
  network connections, and as well as the servlet's class and instance variables. 
  */
  service(ServletRequest, ServletResponse): void

  /**
  Called by the servlet container to indicate to a servlet that the servlet is being taken out of service.

  This method is only called once all threads within the servlet's service method have exited or after a timeout period has passed.
  After the servlet container calls this method, it will not call the service method again on this servlet.
  */
  destroy(): void

  /**
  Returns information about the servlet, such as author, version, and copyright.
  */
  getServletInfo(): String

  /**
  Returns a ServletConfig object, which contains initialization and startup parameters for this servlet.
  */
  getServletConfig(): ServletConfig
}
```

1. (서블릿 컨테이너에 의해) 서블릿이 생성되고, `init()` 메서드 실행
2. (서블릿 컨테이너에 의해) `service()` 메서드 실행 (반드시 `init()` 메서드 성공 이후에 실행)
3. (서블릿 컨테이너에 의해) `destroy()` 메서드 실행 (모든 threads 에서의 처리가 끝났거나, timeout 되었을 때 실행)

<br><br>

### ServletConfig (Interface)

서블릿을 초기화하는 동안, **서블릿 Container -> 서블릿**에게 정보를 전달하기 위해 사용하는 Servlet configuration object 이다.

```java
/**
  A servlet configuration object used by a servlet container to pass information to a servlet during initialization.
  */
public interface ServletConfig {
  getServletName(): String

  getServletContext(): ServletContext

  getInitParameter(String): String

  getInitParameterNames(): Enumeration<String>
}
```

<br><br>

### GenericServlet (AbstractClass)

프로토콜에 독립적인(상관없는) generic 한 서블릿 클래스이다. 즉, 특정 프로토콜을 위해 사용되는 서블릿 클래스가 아니고 한단계 추상적인 의미로 사용될 수 있는 generic 한 서블릿 클래스이다.

```java
/**
Defines a generic, protocol-independent servlet.
*/
public abstract class GenericServlet implements Servlet, ServletConfig, java.io.Serializable {
  init(): void

  init(ServletConfig): void

  /**
  This method is declared abstract so subclasses, such as HttpServlet, must override it.
  */
  service(ServletRequest, ServletResponse): void (abstract)

  destroy(): void
  ...
}
```

Servlet, ServletConfig 인터페이스를 구현했다. (Servlet 기능을 구현)

GenericServlet 을 상속받아 구현한 서블릿들은 사용되는 프로토콜에 따라 service()를 오버라이딩해서 구현한다.

<br><br>

### HttpServlet (AbstractClass)

GenericServlet 을 상속받아 HTTP 프로토콜을 사용하는 서블릿 기능을 수행한다.

위의 GenericServlet 클래스는 말 그대로 generic 한 클래스이다. **웹 (프로토콜) 전용으로 사용하기 위한 Servlet 이 바로 HttpServlet 클래스이다. (웹 서비스를 제공하는 서블릿을 만들 때 사용한다.)**

클라이언트의 요청 시, service() 가 호출되고 클라이언트의 요청 방식에 따라 doGet(), doPost(), doPut() 등의 메소드가 호출된다.


```java
/**
Provides an abstract class to be subclassed to create an HTTP servlet suitable for a Web site.
A subclass of HttpServlet must override at least one method, usually one of these:
- doGet, if the servlet supports HTTP GET requests
- doPost, for HTTP POST requests
- doPut, for HTTP PUT requests
- doDelete, for HTTP DELETE requests
- init and destroy, to manage resources that are held for the life of the servlet
- getServletInfo, which the servlet uses to provide information about itself

There's almost no reason to override the service method.
service handles standard HTTP requests by dispatching them to the handler methods for each HTTP request type (the doMethod methods listed above).
*/
public abstract class HttpServlet extends GenericServlet {
  ...

  doGet(HttpServletRequest, HttpServletResponse): void

  doPost(HttpServletRequest, HttpServletResponse): void

  doPut(HttpServletRequest, HttpServletResponse): void

  doDelete(HttpServletRequest, HttpServletResponse): void

  doHead(HttpServletRequest, HttpServletResponse): void

  doOptions(HttpServletRequest, HttpServletResponse): void

  doTrace(HttpServletRequest, HttpServletResponse): void

  /**
  Dispatches client requests to the protected service method. There's no need to override this method.

  1. 형변환 : Servlet -> HttpServlet
    - ServletRequest -> HttpServletRequest
    - ServletResponse -> HttpServletResponse
  2. service(HttpServletRequest, HttpServletResponse) 호출
  */
  service(ServletRequest, ServletResponse): void

  /**
  Receives standard HTTP requests from the public service method and dispatches them to the doMethod methods defined in this class.

  This method is an HTTP-specific version of the javax.servlet.Servlet.service method. 
  There's no need to override this method.

  1. HTTP 요청(GET, POST, ...)에 따라 doMethod() 호출
  */
  service(HttpServletRequest, HttpServletResponse): void (protected)
}
```

주석에도 나와있듯이, service() 메소드는 오버라이딩할 이유가 거의 없다. 표준 HTTP 요청에 대해서, HTTP 요청을 각각의 핸들러 메소드(doMethod()) 로 보내는 메소드이기 때문이다.

<br><br>

>Reference
> 1. https://mangkyu.tistory.com/14