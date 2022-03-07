**서블릿 컨테이너(e.g. 톰캣) 자체적으로 세션 관리**

<br>

### Session 

**주요 인터페이스/클래스**

- `HttpSession`
- `StandardSession`

```java
/**
 * Provides a way to identify a user across more than one page request or visit to a Web site and to store information about that user.
 * The session persists for a specified time period, across more than one connection or page request from the user. 
 * A session usually corresponds to one user, who may visit a site many times. 
 *
 * ...
 * Session information is scoped only to the current web application ( ServletContext), so information stored in one context will not be directly visible in another.
 */
public interface HttpSession {
    ...
}
```

```java
/**
 * Standard implementation of the Session interface.
 * ...
 */
public class StandardSession implements HttpSession, Session, Serializable {

    ...


    /**
     * The session identifier of this Session.
     */
    protected String id = null;

    /**
     * The collection of user data attributes associated with this Session.
     */
    protected ConcurrentMap<String, Object> attributes = new ConcurrentHashMap<>();

    /**
     * The Manager with which this Session is associated.
     */
    protected transient Manager manager = null;

    ...

}
```

<br>

1. Session(StandardSession) 사용자 데이터 관리 : `ConcurrentMap<String, Object> attributes`
2. 서블릿 컨테이너 입장에서 관리 : Map<SessionId(JSESSIONID), HttpSession> 일듯 
   1. ManagerBase `protected Map<String, Session> sessions = new ConcurrentHashMap<>();` 이거 인듯

<br>

---

### Manager

(위의 StandardSession 중) `Manager manager`

- `Manager`
- `ManagerBase`

```java

/**
 * A Manager manages the pool of Sessions that are associated with a particular Context.
 *
 * Different Manager implementations may support
 * value-added features such as the persistent storage of session data,
 * as well as migrating sessions for distributable web applications.
 *
 * In order for a Manager implementation to successfully operate with a Context implementation
 * that implements reloading, it must obey the following constraints:
 *
 * 1. Must implement Lifecycle so that the Context can indicate that a restart is required.
 * 2. Must allow a call to stop() to be followed by a call to start() on the same Manager instance.
 *
 * @author Craig R. McClanahan
 */
public interface Manager {

    /**
     * Get the Context with which this Manager is associated.
     * 얘가 ServletContext.
     */
    public Context getContext(); // A Context is a Container that represents a servlet context, and therefore an individual web application, in the Catalina servlet engine. 

    ...

```


```java
/**
 * Minimal implementation of the Manager interface that supports no session persistence or distributable capabilities. 
 * This class may be subclassed to create more sophisticated Manager implementations.
 */
public abstract class ManagerBase extends LifecycleMBeanBase implements Manager {

    ...

    /**
     * The name of the algorithm to use to create instances of java.security.SecureRandom which are used to generate session IDs
     * Session ID 생성하기 위해 알고리즘 사용함.
     * PRNG : pseudo-random number generator
     */
    protected String secureRandomAlgorithm = "SHA1PRNG";

    protected SessionIdGenerator sessionIdGenerator = null; // 얘가 secureRandomAlgorithm 사용함

    ...

    /**
     * The set of currently active Sessions for this Manager, keyed by session identifier.
     */
    protected Map<String, Session> sessions = new ConcurrentHashMap<>(); // 중요

    ...
    
    /**
     * Generate and return a new session identifier.
     */
    protected String generateSessionId() {

        String result = null;

        do {
            if (result != null) {
                // Not thread-safe but if one of multiple increments is lost
                // that is not a big deal since the fact that there was any
                // duplicate is a much bigger issue.
                duplicates++;
            }

            result = sessionIdGenerator.generateSessionId();

        } while (sessions.containsKey(result));

        return result;
    }

    ...
}
```

<br>

1. Session ID 생성 시 secure 알고리즘 사용
   1. 랜덤하게 생성하기 위해서 인줄 알았는데, 아래와 같은 내용이 있다./
   2. > " 하지만, 암호학적으로 안전한 난수생성기를 쓰지 않는 경우, 사용자나 공격자들이 무식하게 경우의 수를 전부 미리 계산하고 그 결과를 이용해서 Bruteforce 하게 세션값을 위조해버리거나 조작해버리는 경우도 생길 수 있습니다. " <br><br> 출처 : https://semtax.tistory.com/92
      1. 예를 들어 1, 2, 3, 4, ... 와 같이 세션이 추측 가능하다면, 세션 ID 맘대로 사용 가능해지게 될 듯

2. 세션ID 생성 시 do-while 문 통해서 중복되지 않게 만듬 (`Map.contains()` 체크)


<br><br>

---

### 'JSESSION' 이라는 것은 어디서 가져오는지 ?

```java
public class SessionConfig {

    private static final String DEFAULT_SESSION_COOKIE_NAME = "JSESSIONID";
    private static final String DEFAULT_SESSION_PARAMETER_NAME = "jsessionid";
    
    ...

    public static String getSessionUriParamName(Context context) {

        String result = getConfiguredSessionCookieName(context);

        if (result == null) {
            result = DEFAULT_SESSION_PARAMETER_NAME;
        }

        return result;
    }

    ...
```


<br><br>

---


### 확인해볼것

- `ApplicationSessionCookieConfig`
- `TomcatServletWebServerFactory`

```java
public class TomcatServletWebServerFactory extends AbstractServletWebServerFactory
		implements ConfigurableTomcatWebServerFactory, ResourceLoaderAware {

    ...

	private void configureCookieProcessor(Context context) {
		SameSite sessionSameSite = getSession().getCookie().getSameSite();
		List<CookieSameSiteSupplier> suppliers = new ArrayList<>();
		if (sessionSameSite != null) {
			suppliers.add(CookieSameSiteSupplier.of(sessionSameSite)
					.whenHasName(() -> SessionConfig.getSessionCookieName(context)));
		}
		if (!CollectionUtils.isEmpty(getCookieSameSiteSuppliers())) {
			suppliers.addAll(getCookieSameSiteSuppliers());
		}
		if (!suppliers.isEmpty()) {
			context.setCookieProcessor(new SuppliedSameSiteCookieProcessor(suppliers));
		}
	}
```



```java
public class ApplicationSessionCookieConfig implements SessionCookieConfig {

    ...

    public static Cookie createSessionCookie(Context context,
            String sessionId, boolean secure) {

        SessionCookieConfig scc =
            context.getServletContext().getSessionCookieConfig();

        // NOTE: The priority order for session cookie configuration is:
        //       1. Context level configuration
        //       2. Values from SessionCookieConfig
        //       3. Defaults

        Cookie cookie = new Cookie(
                SessionConfig.getSessionCookieName(context), sessionId);

        // Just apply the defaults.
        cookie.setMaxAge(scc.getMaxAge());
        cookie.setComment(scc.getComment());

        if (context.getSessionCookieDomain() == null) {
            // Avoid possible NPE
            if (scc.getDomain() != null) {
                cookie.setDomain(scc.getDomain());
            }
        } else {
            cookie.setDomain(context.getSessionCookieDomain());
        }

        // Always set secure if the request is secure
        if (scc.isSecure() || secure) {
            cookie.setSecure(true);
        }

        // Always set httpOnly if the context is configured for that
        if (scc.isHttpOnly() || context.getUseHttpOnly()) {
            cookie.setHttpOnly(true);
        }

        cookie.setPath(SessionConfig.getSessionCookiePath(context));

        return cookie;
    }

    ...
}
```

https://semtax.tistory.com/92