## [Servlet Authentication Architecture](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html)

|클래스|설명|
|-|-|
|`SecurityContextHolder`|Where Spring Security stores the details of who is authenticated|
|`SecurityContext`|It is contained in SecurityContextHolder<br><br>It contains the `Authentication` of the currently authenticated user|
|`AuthenticationManager`|'인증'을 처리하기 위한 인터페이스<br><br>`Authentication authenticate(Authentication authentication) throws AuthenticationException;`|
|`ProviderManager`|`AuthenticationManager` 구현 클래스|
|`AuthenticationProvider`|(실질적인) '인증' 을 처리하는 인터페이스<br><br>e.g. `AbstractUserDetailsAuthenticationProvider` : ID, Password 인증<br>(ID, Password 인증은 추상메서드로 되어 있기에, 사용자가 구현)||`Authentication`|`AuthenticationManager.authenticate(Authentication authentication)`<br><br>인증(authentication)을 위한 토큰<br><br>인증(authentication)이 된 주체에 대한 토큰|
|`GrantedAuthority`|(인증 시) 부여된 권한|
|`AbstractAuthenticationProcessingFilter`|A base `Filter` used for authentication<br><br>|


<br><br>

## [SecurityContextHolder](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html#servlet-authentication-securitycontextholder)

<img src="https://docs.spring.io/spring-security/reference/_images/servlet/authentication/architecture/securitycontextholder.png">

(Default) `ThreadLocal` 기반으로 `SecurityContext` 를 가지고 있다.

- FilterChainProxy 가 Cleared SecurityContext 를 보장한다.
  - `SecurityContextHolder.clearContext()`

**전략** 
- MODE_GLOBAL
- MODE_INHERITABLETHREADLOCAL
- MODE_THREADLOCAL


```java
if (strategyName.equals(MODE_THREADLOCAL)) {
    strategy = new ThreadLocalSecurityContextHolderStrategy();
    // private static final ThreadLocal<SecurityContext> contextHolder = new ThreadLocal<>();
}
else if (strategyName.equals(MODE_INHERITABLETHREADLOCAL)) {
    strategy = new InheritableThreadLocalSecurityContextHolderStrategy();
    // private static final ThreadLocal<SecurityContext> contextHolder = new InheritableThreadLocal<>();
}
else if (strategyName.equals(MODE_GLOBAL)) {
    strategy = new GlobalSecurityContextHolderStrategy();
    // private static SecurityContext contextHolder;
}
```

<br><br>

## [SecurityContext](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html#servlet-authentication-securitycontext)

Authentication 객체를 포함한다.

<br><br>

## [Authentication](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html#servlet-authentication-authentication)

**구성 요소**
- principal : 인증 주체 ID
- credentials : 인증 증명을 위한 credentials
  - (대부분) 인증 후에 credential 정보는 지울 수 있도록 한다.
  - e.g. password
- authorities : 인증 주체 권한
  - `GrantAuthority` 상속한 클래스
  - e.g. roles, scopes 등

<br>

**1. 인증하기 위한 객체로 사용될 때**

- isAuthenticated() : false

<br>

**2. 인증된 객체로 사용될 때**

- isAuthenticated() : true
- SecurityContext 로 부터 획득 가능

<br><br>

## [AuthenticationManager](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html#servlet-authentication-authenticationmanager)

인증 처리 위한 API (인터페이스)

<br><br>

## [ProviderManager](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html#servlet-authentication-providermanager)

- AuthenticationManager 의 구현체

- AuthenticationProvider(s) 에게 인증 위임한다.


```java
public class ProviderManager implements AuthenticationManager, MessageSourceAware, InitializingBean {
    ...

	private List<AuthenticationProvider> providers = Collections.emptyList();

    ...

	private AuthenticationManager parent;
```

- (인증을 처리할 수 있는)AuthenticationProvider 가 없는 경우, `ProviderNotFoundException` (AuthenticationException 의 하위 클래스)

```java
public class ProviderNotFoundException extends AuthenticationException {
    ...
}
```

<br><br>

## [AuthenticationProvider](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html#servlet-authentication-authenticationprovider)

실질적인 인증(authentication)을 수행한다.

<br><br>

### 참고

1. https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html