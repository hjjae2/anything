## Dispatcher Servlet

(웹 상에서) 클라이언트로부터 어떠한 요청이 들어오면 (Tomcat 과 같은) Servlet Container 가 요청을 받는다. **이때 제일 앞에서 서버로 들어오는 모든 요청을 처리하는(수신하는) 'Front-Controller' 를 Dispatcher-Servlet 이라고 한다.**

> MVC 아키텍쳐는 보통 이 Front-Controller 패턴과 함께 사용된다고 한다.

공통적인 작업은 Dispatcher-Servlet 이 처리하고 세부적인 작업에 대해서는 적절한 Controller 에 작업을 위임한다.

잠시 Servlet 개념으로 돌아가보면, 기존에는 모든 Servlet 에 대해 web.xml 에서 URL 매핑을 등록해주어야 했다. 그런데 Dispatcher-Servlet 이 등장하면서 해당 애플리케이션으로 들어오는 모든 요청을 핸들링 해주었다. web.xml의 역할을 축소시키고 편리함을 제공했다.

즉, 1. 모든 요청을 한 곳(DispatcherServlet)에서 받아서 필요한 공통의 작업을 처리하고 2. 요청에 맞는 handler(Controller) 로 위임(dispatch), 3. 해당 handler의 실행 결과를 Http Response 형태로 만들어 반환하는 역할을 한다.

> " Spring이 없는 JAVA 런타임에는 Controller 가 존재하지 않는다. 따라서 우리는 서블릿 객체를 생성하고, 그것을 web.xml 에 모두 등록해줘야 했다. "

<br>

1. Controller 의 등장<br>
   `@Controller` 어노테이션을 사용할 수 있게 되었다.
2. Front-Contrller 패턴 구조(2차 Controller 구조)<br>
   Before: 요청 -> web.xml (각각의 서블릿(Controller))<br>
    - web.xml 에 각 Controller 를 모두 등록해줘야 한다.<br>
  
   After: 요청 -> DispatcherServlet (1차) -> 일반 Controller (2차)<br>
    - DispatcherServlet이 모든 요청을 받고, 위임하는 역할을 함으로써 web.xml 에 모두 등록할 필요가 없어졌다.
3. 공통/최우선 작업 처리<br>
   공통적으로 진행되어야 할 작업들을 우선적으로 처리할 수 있다. (인코딩과 같은 것들이 있다.)

<br>

**실행 흐름**

1. **Client -> DispatcherServlet**<br>
   클라이언트가 자원을 요청한다.
2. **DispatcherServlet -> HandlerMapping**<br>
   해당 요청을 처리할 Controller 가 있는지 검색한다.
3. **HandlerMapping -> (특정) Controller**<br>
   Controller 를 찾았으면, 처리를 요청한다.
4. **Controller -> DispatcherServlet**<br>
   요청을 처리하고, 결과를 출력할 View(이름) 를 리턴해준다.
5. **DispatcherServlet -> ViewResolver**<br>
   받은 View(이름)을 검색한다.
6. **ViewResolver -> (해당하는) View**<br>
   ~~해당 View 를 찾았으면, 해당 View 에 Controller 가 처리한 데이터를 전달한다. (?)~~<br>
   해당 View 를 매핑해준다(찾아준다).
7. **View -> DispatcherServlet**<br>
   최종적으로 페이지를 만들고 DispatcherServlet 에 전달한다.
8. **DispatcherServlet -> Client (or Web Server)**<br>
   결과물을 응답한다.
  
<img src="https://user-images.githubusercontent.com/35790290/109492994-2d31f600-7acf-11eb-8ace-250d73bb30d2.png" width="80%" height="80%">

> 출처: https://bk-investing.tistory.com/57?category=903513


<br>

위의 흐름은 효율적으로 보여지나, 실제로는 아래와 같은 문제점이 있었다고 한다.

- DispatcherServlet 는 모든 요청을 처리(수신)하다보니, 이미지/HTMl 파일 등의 정적 파일들에 대한 요청도 전부 Controller 로 넘긴다.
- (DispatcherServlet가 수신하지 않으면 자원을 내어줄 수 있는 상황에서) JSP 파일 안의 JS, CSS 파일들에 대한 요청도 가로채어 수신하기 때문에 자원을 내어주지 못하는 상황이 생긴다. (이것을 처리하는 Controlle 가 없기 때문일 것이다.)

<br>

**해결책 1**
- /apps 의 URL 로 접근하면 DispatcherServlet 가 처리한다.
- /resources 의 URL 로 접근하면 DispatcherServlet 가 처리하지 않는다.

이 방법은 괜찮지만 코드가 지저분해지는 단점이 있다. 

또, 모든 요청에 대해 `/apps`, `/resources` 와 같은 URL 을 붙여주어야 하는 단점이 있다.

<br>

**해결책 2**
- 모든 요청을 컨트롤러에 등록한다. 

무식한 방법이라고 한다.

<br>

위의 문제에 대한 Spring 이 해결책을 제공해준다.

`<map:resources />` 

DispatcherServlet 에서 요청에 대한 Controller 를 찾을 수 없는 경우에, 2차적으로 설정된 경로에 요청을 보내는 것이다.


<br><br>

### 코드로 살펴보기

```java
public class DispatcherServlet extends FrameworkServlet {
}

public abstract class FrameworkServlet extends HttpServletBean implements ApplicationContextAware {
}

public abstract class HttpServletBean extends HttpServlet implements EnvironmentCapable, EnvironmentAware {
}
```




<br><br>

>Reference
>1. https://medium.com/@fntldpf12/dispatcher-servlet%EC%9D%B4%EB%9E%80-624a2195d38f
> 2. https://velog.io/@seculoper235/2.-DispatcherServlet-%EC%9D%B4%EB%9E%80
> 3. https://galid1.tistory.com/525
