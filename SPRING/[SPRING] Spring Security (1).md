## Overview

![Spring Security Authentication Architecture](https://user-images.githubusercontent.com/35790290/151847650-5846666b-6e94-4967-8d59-b8733882d00b.png)

<br><br>

## [A Review Of Filters](https://docs.spring.io/spring-security/reference/servlet/architecture.html#servlet-filters-review)

<br><br>

## [DelegatingFilterProxy](https://docs.spring.io/spring-security/reference/servlet/architecture.html#servlet-delegatingfilterproxy)

> package org.springframework.web.filter;

DelegatingFilterProxy 는 서블릿 컨테이너(Servet Conatiner)와 스프링 애플리케이션컨텍스트(ApplicationContext) 사이를 연결시켜주는(bridge) 역할을 한다.

- 서블릿 컨테이너는 스프링에 등록된 Bean 들을 사용(인식)하지 않는다.
- 따라서, (서블릿 컨테이너 표준에 맞게 등록된) DelegatingFilterProxy가 스프링에 등록된 Bean(단, Fiter 구현체)에 위임한다.

> "(Servlet)Filter vs Bean Filter" 의 차이에 대해 이해할 것

<br>

```java
@Override
public void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain) throws ServletException, IOException {

	// Lazily initialize the delegate if necessary.
	Filter delegateToUse = this.delegate;
	if (delegateToUse == null) {
		synchronized (this.delegateMonitor) {
			delegateToUse = this.delegate;
			if (delegateToUse == null) {
				WebApplicationContext wac = findWebApplicationContext();
				if (wac == null) {
					throw new IllegalStateException("No WebApplicationContext found: " +
							"no ContextLoaderListener or DispatcherServlet registered?");
				}
				delegateToUse = initDelegate(wac); // WebApplicationContext 로 부터 위임할 Filter 를 가져온다(찾는다).
			}
			this.delegate = delegateToUse;
		}
	}

	// Let the delegate perform the actual doFilter operation.
	invokeDelegate(delegateToUse, request, response, filterChain); // work를 위임한다.
}
```

<br><br>

## [FilterChainProxy](https://docs.spring.io/spring-security/reference/servlet/architecture.html#servlet-filterchainproxy)

> package org.springframework.security.web;

<br>

> Spring Security’s Servlet support is contained within FilterChainProxy
>
> FilterChainProxy is a special Filter provided by Spring Security that allows delegating to many Filter instances through SecurityFilterChain

- (Spring Security에서)`DelegatingFilterProxy`가 security work를 위해 위임하는 Filter(Bean Filter)이다.

- `FilterChainProxy`는 Spring Security를 지원하기 위해 사용된다.

- FilterChainProxy가 `SecurityFilterChain`의 다양한 Filter에 (work를)위임한다.


<img src=https://docs.spring.io/spring-security/reference/_images/servlet/architecture/filterchainproxy.png>

> 출처 : https://docs.spring.io/spring-security/reference/servlet/architecture.html

<br><br>

## [SecurityFilterChain](https://docs.spring.io/spring-security/reference/servlet/architecture.html#servlet-securityfilterchain)

> package org.springframework.security.web;

> SecurityFilterChain is used by FilterChainProxy to determine which Spring Security Filters should be invoked for this request.

```java
// FilterChainProxy.java

private List<Filter> getFilters(HttpServletRequest request) {
	for (SecurityFilterChain chain : filterChains) {
		if (chain.matches(request)) {
			return chain.getFilters();
		}
	}
	return null;
}

...


private void doFilterInternal(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
	FirewalledRequest fwRequest = firewall.getFirewalledRequest((HttpServletRequest) request);
	HttpServletResponse fwResponse = firewall.getFirewalledResponse((HttpServletResponse) response);

	List<Filter> filters = getFilters(fwRequest);
	if (filters == null || filters.size() == 0) {
		...
		fwRequest.reset();
		chain.doFilter(fwRequest, fwResponse);
		return;
	}

	VirtualFilterChain vfc = new VirtualFilterChain(fwRequest, chain, filters);
	vfc.doFilter(fwRequest, fwResponse);
}
```


<img src="https://docs.spring.io/spring-security/reference/_images/servlet/architecture/securityfilterchain.png">

<br><br>

<img src="https://docs.spring.io/spring-security/reference/_images/servlet/architecture/multi-securityfilterchain.png">



<br>
<br>

(In SecurityFilterChain)Security Fiter 들은 (DelegatingFilterProxy가 아닌)FilterChainProxy에 등록(관리)된다.

이를 통해 얻는 이점은 다음과 같은 것들이 있다.

1. Spring Security 관련 디버깅 시, FilterChainProxy를 starting point 로 사용할 수 있다.
2. Spring Security 관련된 로직을 추가하고 싶다면, FilterChainProxy에 추가하면 된다. (무조건 실행되는, 핵심적인 부분이기 때문에)

<br>

추가적인 특징으로는 다음과 같은 것들이 있다.

- SecurityFilterChain 은 Security Filter 를 0~n 개 가질 수 있다. 
- 또, 독립적인 여러 개의 SecurityFilterChain을 구성할 수 있다.

<Br><Br>

## [Security Filters](https://docs.spring.io/spring-security/reference/servlet/architecture.html#servlet-security-filters)

Security Filter 의 종류는 대표적으로 다음과 같은 것들이 있다.

- UsernamePasswordAuthenticationFilter 
- BasicAuthenticationFilter
- ...
- ExceptionTranslationFilter
- FilterSecurityInterceptor
- ...

<br><br>

## [Handling Security Exceptions](https://docs.spring.io/spring-security/reference/servlet/architecture.html#servlet-security-filters)

`ExceptionTranslationFilter`는 AccessDeniedException / AuthenticationException 을 HTTP Response 로 변환한다.

```java
try {
	filterChain.doFilter(request, response); 
} catch (AccessDeniedException | AuthenticationException ex) {
	if (!authenticated || ex instanceof AuthenticationException) {
		startAuthentication(); 
	} else {
		accessDenied(); 
	}
}
```

```java
protected void sendStartAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain, AuthenticationException reason) throws ServletException, IOException {
	// SEC-112: Clear the SecurityContextHolder's Authentication, as the
	// existing Authentication is no longer considered valid
	SecurityContextHolder.getContext().setAuthentication(null);
	requestCache.saveRequest(request, response);
	logger.debug("Calling Authentication entry point.");
	authenticationEntryPoint.commence(request, response, reason);
}


private void handleSpringSecurityException(HttpServletRequest request, HttpServletResponse response, FilterChain chain, RuntimeException exception) throws IOException, ServletException {
	if (exception instanceof AuthenticationException) {
		logger.debug("Authentication exception occurred; redirecting to authentication entry point", exception);

		sendStartAuthentication(request, response, chain, (AuthenticationException) exception);
	}
	else if (exception instanceof AccessDeniedException) {
		Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
		if (authenticationTrustResolver.isAnonymous(authentication) || authenticationTrustResolver.isRememberMe(authentication)) {
			logger.debug("Access is denied (user is " + (authenticationTrustResolver.isAnonymous(authentication) ? "anonymous" : "not fully uthenticated") + "); redirecting to authentication entry point", exception);

			sendStartAuthentication(
					request,
					response,
					chain,
					new InsufficientAuthenticationException(messages.getMessage("ExceptionTranslationFilter.insufficientAuthentication", "Full authentication is required to access this resource"))
            );
		}
		else {
			logger.debug("Access is denied (user is not anonymous); delegating to AccessDeniedHandler", exception);

			accessDeniedHandler.handle(request, response, (AccessDeniedException) exception);
		}
	}
}
```

> If the application does not throw an AccessDeniedException or an AuthenticationException, then ExceptionTranslationFilter does not do anything.

위 코드를 보면, 아래와 같이 호출되는 것을 알 수 있다.

- `AuthenticationException` -> `authenticationEntryPoint.commence()` 
- `AccessDeniedException` -> `accessDeniedHandler.handle()`
  

그렇기에 AuthenticationEntryPoint, AccessDeniedHandler 를 재정의(상속)하여 커스텀하게 핸들링할 수 있다.<br>
(error response, redirect to entry point 등)

```java
...

http.exceptionHandling().accessDeniedHandler(new GPAccessDeniedHandler());
http.exceptionHandling().authenticationEntryPoint(new GPAuthenticationEntryPoint());

...
```

<br><br>

### 참고

1. https://docs.spring.io/spring-security/reference/servlet/architecture.html
2. https://mangkyu.tistory.com/76
