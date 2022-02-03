## ServletContextListener

ServletContext 의 변화(라이프사이클)를 감지한다.
- 웹 애플리케이션의 '시작', '종료' 시에 아래 메서드가 호출된다.
  - `contextInitialized()` : 웹 애플리케이션이 시작될 때 호출
    - servlet, filter 의 초기화보다 먼저 실행
  - `contextDestroyed()` : 웹 애플리케이션이 종료될 때 호출
    - servlet, filter 는 해당 메서드 호출 전에 이미 종료된 상태


```java
/**
 * Implementations of this interface receive notifications about changes to the
 * servlet context of the web application they are part of. To receive
 * notification events, the implementation class must be configured in the
 * deployment descriptor for the web application.
 *
 * @see ServletContextEvent
 * @since v 2.3
 */

public interface ServletContextListener extends EventListener {

    /**
     ** Notification that the web application initialization process is starting.
     * All ServletContextListeners are notified of context initialization before
     * any filter or servlet in the web application is initialized.
     * The default implementation is a NO-OP.
     * @param sce Information about the ServletContext that was initialized
     */
    public default void contextInitialized(ServletContextEvent sce) {
    }

    /**
     ** Notification that the servlet context is about to be shut down. All
     * servlets and filters have been destroyed before any
     * ServletContextListeners are notified of context destruction.
     * The default implementation is a NO-OP.
     * @param sce Information about the ServletContext that was destroyed
     */
    public default void contextDestroyed(ServletContextEvent sce) {
    }
}
```

<br><br>

## ContextLoader

**(Root)ApplicationContext 를 위해 실질적인 초기화 작업을 수행한다.**
- **ServletContext 에 WebApplicationContext 를 설정**
- **WebApplicationContext 에 ServletContext 를 설정**


```java
servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, this.context);

wac.setServletContext(sc);
```

<br>

- ContextLoaderListner 에 의해 호출된다.
  - contextInitialized()
  - contextDestroyed()

<br>

```java
/**
 * Performs the actual initialization work for the root application context.
 * Called by {@link ContextLoaderListener}.
 *
 * <p>Looks for a "contextClass" parameter at the
 * web.xml context-param level to specify the context class type, falling
 * back to XmlWebApplicationContext if not found. 
 * With the default ContextLoader implementation, any context class
 * specified needs to implement the ConfigurableWebApplicationContext interface.
 */
public class ContextLoader {
    ...

    public WebApplicationContext initWebApplicationContext(ServletContext servletContext) {
        
		long startTime = System.currentTimeMillis();

		try {
			// Store context in local instance variable, to guarantee that
			// it is available on ServletContext shutdown.
			if (this.context == null) {
				this.context = createWebApplicationContext(servletContext);
			}
			if (this.context instanceof ConfigurableWebApplicationContext) {
				ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext) this.context;
				if (!cwac.isActive()) {
					// The context has not yet been refreshed -> provide services such as
					// setting the parent context, setting the application context id, etc
					if (cwac.getParent() == null) {
						// The context instance was injected without an explicit parent ->
						// determine parent for root web application context, if any.
						ApplicationContext parent = loadParentContext(servletContext);
						cwac.setParent(parent);
					}
					configureAndRefreshWebApplicationContext(cwac, servletContext);
				}
			}
			servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, this.context);

            ...


			return this.context;
		}
		catch (RuntimeException | Error ex) {
            ...
		}
	}
}
```

<br><br>

## ContextLoaderListener

**Spring 의 (Root)WebApplicationContext 의 '시작', '종료' 를 위한 리스너.**

(ContextLoader 를 통해)
- **ServletContext 에 WebApplicationContext 를 설정**
- **WebApplicationContext 에 ServletContext 를 설정**

```java
/**
 * Bootstrap listener to start up and shut down Spring's root WebApplicationContext.
 * As of Spring 3.1, ContextLoaderListener supports injecting the root web application context 
 * via the ContextLoaderListener(WebApplicationContext) constructor, allowing for programmatic configuration in Servlet 3.0+ environments. 
 * See org.springframework.web.WebApplicationInitializer for usage examples.
 */
public class ContextLoaderListener extends ContextLoader implements ServletContextListener {

    public ContextLoaderListener() {
	}

    public ContextLoaderListener(WebApplicationContext context) {
		super(context);
	}

    @Override
	public void contextInitialized(ServletContextEvent event) {
        // initWebApplicationContext 는 ContextLoader 메서드이다.
        // ServletContext 에 WebApplicationContext 를 설정한다.
		initWebApplicationContext(event.getServletContext());
	}


    @Override
	public void contextDestroyed(ServletContextEvent event) {
        // closeWebApplicationContext 는 ContextLoader 메서드이다.
		closeWebApplicationContext(event.getServletContext());
		ContextCleanupListener.cleanupAttributes(event.getServletContext());
	}
}
```

ServletContext 가 초기화 되는 시점에, ContextLoaderListner(ContextLoader) 가 ServletContext 에 WebApplicationContext 를 설정합니다.

<br><br>

## ApplicationContext

```java
/**
 * Central interface to provide configuration for an application.
 * This is read-only while the application is running, but may be reloaded if the implementation supports this.
 */
public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory, MessageSource, ApplicationEventPublisher, ResourcePatternResolver {
    ...
}
```

<br>

**참고**

|인터페이스|설명|
|-|-|
|`ListableBeanFactory`|애플리케이션의 컴포넌트들에 접근하기 위한 Bean Factory Method|
|`ResourceLoader`|File resources 를 로드히기 위한 인터페이스|
|`ApplicationEventPublish`|Event publishing 을 위한 인터페이스|
|`MessageSource`|국제화(Internationalization)를 지원하기 위한 인터페이스|

<br><br>

## WebApplicationContext

This interface adds a getServletContext() method to the generic ApplicationContext interface

```java
/**
 * Interface to provide configuration for a web application.
 * This interface adds a getServletContext() method to the generic ApplicationContext interface, 
 * and defines a well-known application attribute name that the root context must be bound to in the bootstrap process.
 * 
 * Like generic application contexts, web application contexts are hierarchical.
 * There is a single root context per application,
 * while each servlet in the application (including a dispatcher servlet in the MVC framework) has its own child context.
 * 
 * In addition to standard application context lifecycle capabilities,
 * WebApplicationContext implementations need to detect ServletContextAware beans 
 * and invoke the setServletContext method accordingly.
 *
 */
public interface WebApplicationContext extends ApplicationContext {
    ...

    @Nullable
	ServletContext getServletContext();
}
```

<br><br>

## DispatcherServlet

스프링에서 정의한 Front Controller Servlet.

- 모든 요청을 받아, 각각의 handler 로 요청을 위임한다.
- Handler 의 결과를 Http Response 형태로 만들어 반환한다.

<img src="https://docs.spring.io/spring-framework/docs/current/reference/html/images/mvc-context-hierarchy.png">

RootWebApplicationContext 는 infrastructure beans(repository, service 등) 를 포함한다. 
- RootWebApplicationContext 는 모든 Servlet 에 공유된다.
- 해당 Bean 들은 ServletWebApplicationContext 에 상속되거나 재정의될 수 있다.

ServletWebApplicationContext 는 controller, viewResolver, handlerMapping 등의 bean을 관리한다.
- ServletWebApplicationContext 는 해당 DispatcherServlet 내에서만 사용할 수 있다.

> 다만, 요새는 위와 같이 RootWebApplicationContext, ServletWebApplicationContext 와 같이 구분짓지 않고 하나의 IoC Container 에서 관리한다고도 한다.