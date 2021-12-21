### Filter

요청 / 응답을 필터링한다.

DispatcherServlet 이전에 실행 된다. 즉, 스프링 영역 외부에 존재하여, 스프링과 무관한 자원에 대해 동작한다.

- init() : ServletContainer 에 Filter 가 등록되어 초기화될 때 실행
- doFilter() : Filter가 적용된 servlet에 요청이 들어왔을 때 실행 (servlet에 전달하기 전에)
- destroy() : ServletContainer 가 종료될 때(Filter 가 삭제될 때)실행

일반적으로 Encoding 처리, XSS 방어 등을 처리하기 위해 사용될 수 있다.

<br>

### Interceptor

요청/응답에 대해 전(pre)/후(post)로 가로챈다.

DispatcherServlet 이 Controller 를 호출하기 전/후로 가로챈다.
- DispatcherServlet -> Handler Mapping -> **Interceptor** -> Controller(Handler)
  
스프링의 모든 Bean (객체)에 접근할 수 있다.

Interceptor 는 여러 개를 사용할 수 있다.

HTTP 헤더, HTTP 메소드, 세션 / 쿠키, 요청 URL 등을 가져올 수 있다.

- preHandler()<br>
  Controller(Handler) 가 실행되기 전에 처리된다.
- postHandler()<br>
  Controller(Handler) 가 실행된 후에 처리된다.
- afterCompletion()<br>
  view 페이지가 렌더링 되고 난 후에 처리된다.

일반적으로 로그인 체크, 권한(JWT), 프로그램 실행시간 계산 등을 처리하기 위해 사용될 수 있다.

> SpringBoot 에서는 HandlerInterceptor 인터페이스를 구현하여 사용할 수 있다.

<br>

### AOP

OOP 시에, 종단면(관점)에서 중복되는 것들을 처리한다.

Filter, Interceptor 와 달리 메소드 전/후에 자유롭게 설정이 가능하다.

AOP <-> HandlerInterceptor 의 가장 큰 차이는 '파라미터의 차이' 이다.
- Advice 의 경우 JoinPoint, ProceedingJoinPoint 등을 사용
- HandlerInterceptor 는 Filter 와 유사하게 HttpServletRequest, HttpServletResponse 를 사용

일반적으로 로깅, 트랜잭션, 에러처리 등의 처리를 위해 사용될 수 있다.

<br>

> Reference
> 1. https://galid1.tistory.com/521?category=783055
> 2. https://bk-investing.tistory.com/61