## Dispatcher Servlet

(웹 상에서) 클라이언트로부터 어떠한 요청이 들어오면 (Tomcat 과 같은) Servlet Container 가 요청을 받는다. **이때 제일 앞에서 서버로 들어오는 모든 요청을 처리하는(수신하는) 'Front-Controller' 를 Dispatcher-Servlet 이라고 한다.**

> MVC 아키텍쳐는 보통 이 Front-Controller 패턴과 함께 사용된다고 한다.

공통적인 작업은 Dispatcher-Servlet 이 처리하고 세부적인 작업에 대해서는 적절한 Controller 에 작업을 위임한다.

> 잠시 Servlet 개념으로 돌아가보면, 기존에는 모든 Servlet 에 대해 web.xml 에서 URL 매핑을 등록해주어야 했다. 그런데 Dispatcher-Servlet 이 등장하면서 해당 애플리케이션으로 들어오는 모든 요청을 핸들링 해주었다. web.xml의 역할을 축소시키고 편리함을 제공했다.

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

> 이 방법은 괜찮지만 코드가 지저분해지는 단점이 있다. 
> 
> 또, 모든 요청에 대해 `/apps`, `/resources` 와 같은 URL 을 붙여주어야 하는 단점이 있다.

<br>

**해결책 2**
- 모든 요청을 컨트롤러에 등록한다.
> 무식한 방법이라고 한다.

<br>

**위의 문제에 대한 Spring 이 해결책을 제공해준다.**

`<map:resources />` 

DispatcherServlet 에서 요청에 대한 Controller 를 찾을 수 없는 경우에, 2차적으로 설정된 경로에 요청을 보내는 것이다.


<br><br>

## 코드로 살펴보기


상속(구현) 관계

DispatcherServlet -> FrameworkServlet -> (HttpServletBean) -> HttpServlet

```java
/**
Central dispatcher for HTTP request handlers/controllers, e.g. for web UI controllers or HTTP-based remote service exporters. 
Dispatches to registered handlers for processing a web request, providing convenient mapping and exception handling facilities.
*/
public class DispatcherServlet extends FrameworkServlet {
}


/**
Base servlet for Spring's web framework.
Provides integration with a Spring application context, in a JavaBean-based overall solution.
*/
public abstract class FrameworkServlet extends HttpServletBean implements ApplicationContextAware {
}


public abstract class HttpServletBean extends HttpServlet implements EnvironmentCapable, EnvironmentAware {
}
```

<br><br>

### 1. ApplicationFilterChain filter 처리 후, `servlet.service(request, response);` 호출

> 몇몇 분기 내용은 배제

```java
public final class ApplicationFilterChain implements FilterChain {

    ...

    private void internalDoFilter(ServletRequest request, ServletResponse response) throws IOException, ServletException {

        // Call the next filter if there is one (다음 Filter 가 있다면, 호출합니다.)
        if (pos < n) {
            ApplicationFilterConfig filterConfig = filters[pos++];
            try {
                Filter filter = filterConfig.getFilter();

                ...  // (중략)

                filter.doFilter(request, response, this);
            } catch (IOException | ServletException | RuntimeException e) {
                ...
            } catch (Throwable e) {
                ...
            }
            return;
        }

        // We fell off the end of the chain -- call the servlet instance
        // (FilterChain 의 끝에 도착 -> Servlet 객체를 호출합니다.)
        try {

            ...  // (중략)
            
            servlet.service(request, response); // <-- 여기
        } catch (IOException | ServletException | RuntimeException e) {
            ...
        } catch (Throwable e) {
            ...
        } finally {
            ...
        }
    }
}
```


<br>

### 2. HttpServlet(Servlet) service() 메서드 호출

- 타입 변환 : ServletRequest -> HttpServletRequest
- doPost(), doGet() 등의 메서드 호출
  - > 현재 테스트에서는 doPost() 호출

```java
public abstract class HttpServlet extends GenericServlet {
   ...

    @Override
    public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException {
        HttpServletRequest  request;
        HttpServletResponse response;

        try {
            request = (HttpServletRequest) req;
            response = (HttpServletResponse) res;
        } catch (ClassCastException e) {
            throw new ServletException(lStrings.getString("http.non_http"));
        }
        service(request, response);
    }

    ...

    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

        String method = req.getMethod();

        if (method.equals(METHOD_GET)) {
            ...
            // If-Modified-Since(Last-Modified) (캐시 분기)
            doGet(req, resp);
            // or resp.setStatus(HttpServletResponse.SC_NOT_MODIFIED);

        } else if (method.equals(METHOD_HEAD)) {
            long lastModified = getLastModified(req);
            maybeSetLastModified(resp, lastModified);
            doHead(req, resp);

        } else if (method.equals(METHOD_POST)) {
            doPost(req, resp);

        } else if (method.equals(METHOD_PUT)) {
            doPut(req, resp);

        } else if (method.equals(METHOD_DELETE)) {
            doDelete(req, resp);

        } else if (method.equals(METHOD_OPTIONS)) {
            doOptions(req,resp);

        } else if (method.equals(METHOD_TRACE)) {
            doTrace(req,resp);

        } else {
            String errMsg = lStrings.getString("http.method_not_implemented");

            ...

            resp.sendError(HttpServletResponse.SC_NOT_IMPLEMENTED, errMsg);
        }
    }   
}
```

<br>

### 3. FrameworkServlet `doPost()`

- FrameworkServlet : `doPost()` 호출
- FrameworkServlet : `processRequest(request, response)` 호출
- DispatcherServlet : `doService()` 호출

```java

public abstract class FrameworkServlet extends HttpServletBean implements ApplicationContextAware {

	...

	@Override
	protected final void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		processRequest(request, response);  // <-- 여기
	}

	// Process this request, publishing an event regardless of the outcome.
	// The actual event handling is performed by the abstract doService template method.
	protected final void processRequest(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

		...

		try {
		    doService(request, response);  // <-- 여기
		}
		catch (ServletException | IOException ex) {
            	...
		}
		catch (Throwable ex) {
            ...
		}

		finally {
			resetContextHolders(request, previousLocaleContext, previousAttributes);
			if (requestAttributes != null) {
				requestAttributes.requestCompleted();
			}
			logResult(request, response, failureCause, asyncManager);
			publishRequestHandledEvent(request, response, startTime, failureCause);
		}
	}   
}
```

<br>

### 4. DispatcherServlet `doService()`

- DispatcherServlet : `doService()` 호출
- DispatcherServlet : `doDsipatch(request, response);` 호출

```java
public class DispatcherServlet extends FrameworkServlet {

    ...

	protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {
		logRequest(request);

		// Keep a snapshot of the request attributes in case of an include,
		// to be able to restore the original attributes after the include.
                // (attributeSnapShow 저장)
		Map<String, Object> attributesSnapshot = null;
		while(...) {
		    attributesSnapshot.put(attrName, request.getAttribute(attrName));
		}

		// Make framework objects available to handlers and view objects.
                // (HttpServletRequest 에 WebApplicationContext, localeResolver 등 저장)
		request.setAttribute(WEB_APPLICATION_CONTEXT_ATTRIBUTE, getWebApplicationContext());
		request.setAttribute(LOCALE_RESOLVER_ATTRIBUTE, this.localeResolver);
		request.setAttribute(THEME_RESOLVER_ATTRIBUTE, this.themeResolver);
		request.setAttribute(THEME_SOURCE_ATTRIBUTE, getThemeSource());

		...

		try {
			doDispatch(request, response);
		}
		finally {
			...
		}
	}

	...
}
```

<br>

### 5. DispatcherServlet `doDispatch()`

> 중요!

(1) HandlerExecutionChain 객체 획득 (`getHandler()`) <br>
(2) HandlerAdapter 객체 획득 (`getHandlerAdapter()`) <br>
(3) 인터셉터의 preHandle() 호출 <br>
(4) handlerAdapter.handle() 호출 (-> ModelAndView 객체 획득) <br>
(5) 인터셉터의 postHandle() 호출 <br>

```java
public class DispatcherServlet extends FrameworkServlet {

	...

	protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
		HttpServletRequest processedRequest = request;
		HandlerExecutionChain mappedHandler = null;
		boolean multipartRequestParsed = false;

		WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

		try {
			ModelAndView mv = null;
			Exception dispatchException = null;

			try {
				// Convert the request into a multipart request, and make multipart resolver available.
				// If no multipart resolver is set, simply use the existing request.
				// (checkMultiPart() 의 경우, multipart 요청이면 MultipartHttpServletRequest 객체를 반환합니다.)
				processedRequest = checkMultipart(request);
				multipartRequestParsed = (processedRequest != request);

				...  // (중략)

				// Determine handler for the current request.
				// (HandlerExecutionChain 객체를 얻습니다.)
				// ( = 우리가 작성한 Controller/Method 를 의미합니다.)
				mappedHandler = getHandler(processedRequest);
				if (mappedHandler == null) {
					noHandlerFound(processedRequest, response);
					return;
				}


				// Determine handler adapter for the current request.
				// (HandlerAdapter 객체를 얻습니다.)
				HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

				// Process last-modified header, if supported by the handler.
				String method = request.getMethod();
				boolean isGet = "GET".equals(method);
				if (isGet || "HEAD".equals(method)) {
					long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
					if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
						return;
					}
				}

				// Apply preHandle methods of registered interceptors. (in HandlerExecutionChain)
				// (등록된 인터셉터의 preHandle 을 실행합니다.)
				if (!mappedHandler.applyPreHandle(processedRequest, response)) {
					return;
				}

				// Actually invoke the handler.
				// (HandlerAdapter handel() 메서드를 호출합니다.)
				// (ModelAndView 를 반환받습니다.)
				mv = ha.handle(processedRequest, response, mappedHandler.getHandler()); // <-- 여기

				if (asyncManager.isConcurrentHandlingStarted()) {
					return;
				}

				applyDefaultViewName(processedRequest, mv);
				// Apply postHandle methods of registered interceptors.
				// (등록된 인터셉터의 postHandle 을 실행합니다.)
				mappedHandler.applyPostHandle(processedRequest, response, mv);
			}
			catch (Exception ex) {
				dispatchException = ex;
			}
			catch (Throwable err) {
				// As of 4.3, we're processing Errors thrown from handler methods as well,
				// making them available for @ExceptionHandler methods and other scenarios.
				dispatchException = new NestedServletException("Handler dispatch failed", err);
			}
			processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
		}
		catch (Exception ex) {
			triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
		}
		catch (Throwable err) {
			triggerAfterCompletion(processedRequest, response, mappedHandler,
					new NestedServletException("Handler processing failed", err));
		}
		finally {
			if (asyncManager.isConcurrentHandlingStarted()) {
				// Instead of postHandle and afterCompletion
				if (mappedHandler != null) {
					mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
				}
			}
			else {
				// Clean up any resources used by a multipart request.
				if (multipartRequestParsed) {
					cleanupMultipart(processedRequest);
				}
			}
		}
	}

	...

	@Nullable
	protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
		if (this.handlerMappings != null) {
			for (HandlerMapping mapping : this.handlerMappings) {
				HandlerExecutionChain handler = mapping.getHandler(request);
				if (handler != null) {
					return handler;
				}
			}
		}
		return null;
	}

	...

	// Return the HandlerAdapter for this handler object.
	protected HandlerAdapter getHandlerAdapter(Object handler) throws ServletException {
		if (this.handlerAdapters != null) {
			for (HandlerAdapter adapter : this.handlerAdapters) {
				if (adapter.supports(handler)) {
					return adapter;
				}
			}
		}
		throw new ServletException("No adapter for handler [" + handler +
				"]: The DispatcherServlet configuration needs to include a HandlerAdapter that supports this handler");
	}
}
```

<br>

### 6. HandlerAdapter.handle() 호출

- AbstractHandlerMethodAdapter : `handle()`
- RequestMappingHandlerAdapter : `handlerInternal()`
  - `invokeHandlerMethod()` 호출

```java
public abstract class AbstractHandlerMethodAdapter extends WebContentGenerator implements HandlerAdapter, Ordered {

	...

	@Override
	@Nullable
	public final ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {

		// RequestMappingHandlerAdapter.handleInternal() 호출
		return handleInternal(request, response, (HandlerMethod) handler);
	}

	...

}
```

```java
public class RequestMappingHandlerAdapter extends AbstractHandlerMethodAdapter {
   
	...

	@Override
	protected ModelAndView handleInternal(HttpServletRequest request, HttpServletResponse response, HandlerMethod handlerMethod) throws Exception {

		ModelAndView mav;
		checkRequest(request); // 세션 필수 여부, HTTP Method 체크

		...  // (중략)

		// Execute invokeHandlerMethod in synchronized block if required.
		if (this.synchronizeOnSession) {
			HttpSession session = request.getSession(false);
			if (session != null) {
				Object mutex = WebUtils.getSessionMutex(session);
				synchronized (mutex) {
					mav = invokeHandlerMethod(request, response, handlerMethod);
				}
			}
			else {
				// No HttpSession available -> no mutex necessary
				mav = invokeHandlerMethod(request, response, handlerMethod);
			}
		}
		else {
			// No synchronization on session demanded at all...
			mav = invokeHandlerMethod(request, response, handlerMethod);
		}

		if (!response.containsHeader(HEADER_CACHE_CONTROL)) {
			if (getSessionAttributesHandler(handlerMethod).hasSessionAttributes()) {
				applyCacheSeconds(response, this.cacheSecondsForSessionAttributeHandlers);
			}
			else {
				prepareResponse(response);
			}
		}

		return mav;
	}

	...
}
```

<br>


### 7. HandlerAdapter : invokeHandlerMethod() 호출

- `invocableMethod.invokeAndHandle(webRequest, mavContainer);` 호출

```java
public class RequestMappingHandlerAdapter extends AbstractHandlerMethodAdapter {
   
	...

	@Nullable
	protected ModelAndView invokeHandlerMethod(HttpServletRequest request, HttpServletResponse response, HandlerMethod handlerMethod) throws Exception {

		ServletWebRequest webRequest = new ServletWebRequest(request, response);
		try {

			ServletInvocableHandlerMethod invocableMethod = createInvocableHandlerMethod(handlerMethod); // <-- 여기

			...  // (중략 : asyncManager, mavContainer 등의 설정)

			// Invoke the method and handle the return value 
			// through one of the configured HandlerMethodReturnValueHandlers.
			invocableMethod.invokeAndHandle(webRequest, mavContainer);

			if (asyncManager.isConcurrentHandlingStarted()) {
				return null;
			}

			return getModelAndView(mavContainer, modelFactory, webRequest);
		}
		finally {
			webRequest.requestCompleted();
		}
	}

	...
}
```

<br>

### 8. ServletInvocableHandlerMethod :: invokeForRequest() 호출

(1) `Object returnValue = invokeForRequest(webRequest, mavContainer, providedArgs);` <br>
(2) `this.returnValueHandlers.handleReturnValue(returnValue, getReturnValueType(returnValue), mavContainer, webRequest);`

```java
public class ServletInvocableHandlerMethod extends InvocableHandlerMethod {
	...

	// Invoke the method and handle the return value 
	// through one of the configured HandlerMethodReturnValueHandlers.
	public void invokeAndHandle(ServletWebRequest webRequest, ModelAndViewContainer mavContainer, Object... providedArgs) throws Exception {

		Object returnValue = invokeForRequest(webRequest, mavContainer, providedArgs);
		setResponseStatus(webRequest);

		if (returnValue == null) {
			if (isRequestNotModified(webRequest) || getResponseStatus() != null || mavContainer.isRequestHandled()) {
				disableContentCachingIfNecessary(webRequest);
				mavContainer.setRequestHandled(true);
				return;
			}
		}
		else if (StringUtils.hasText(getResponseStatusReason())) {
			mavContainer.setRequestHandled(true);
			return;
		}

		mavContainer.setRequestHandled(false);
		Assert.state(this.returnValueHandlers != null, "No return value handlers");
		try {
			this.returnValueHandlers.handleReturnValue(
					returnValue, getReturnValueType(returnValue), mavContainer, webRequest);
		}
		catch (Exception ex) {
			if (logger.isTraceEnabled()) {
				logger.trace(formatErrorForReturnValue(returnValue), ex);
			}
			throw ex;
		}
	}

	...
}
```


<br>

### 9. ServletInvocableHandlerMethod :: invokeAndHandle() 호출

(1) getMethodArgumentValues() : Object -> DTO 변환<br>
   - HandlerMethodArgumentResolver <br>

(2) doInvoke(args);

```text
resolvers = {HandlerMethodArgumentResolverComposite@16108} 
- argumentResolvers = {LinkedList@16583}  size = 36
  0 = {ProxyingHandlerMethodArgumentResolver@16599} 
  1 = {RequestParamMethodArgumentResolver@16600} 
  2 = {RequestParamMapMethodArgumentResolver@16601} 
  3 = {PathVariableMethodArgumentResolver@16602} 
  4 = {PathVariableMapMethodArgumentResolver@16603} 
  5 = {MatrixVariableMethodArgumentResolver@16604} 
  6 = {MatrixVariableMapMethodArgumentResolver@16605} 
  7 = {ServletModelAttributeMethodProcessor@16606} 
  8 = {RequestResponseBodyMethodProcessor@16589} 
  9 = {RequestPartMethodArgumentResolver@16607} 
  ...
```

```java
public class InvocableHandlerMethod extends HandlerMethod {

	...   

	private HandlerMethodArgumentResolverComposite resolvers = new HandlerMethodArgumentResolverComposite();
	// private final List<HandlerMethodArgumentResolver> argumentResolvers = new LinkedList<>();
	// private final Map<MethodParameter, HandlerMethodArgumentResolver> argumentResolverCache = new ConcurrentHashMap<>(256);

	...   

	@Nullable
	public Object invokeForRequest(NativeWebRequest request, @Nullable ModelAndViewContainer mavContainer, Object... providedArgs) throws Exception {

		// 여기서 Object -> DTO 로 변환
		Object[] args = getMethodArgumentValues(request, mavContainer, providedArgs); 
		if (logger.isTraceEnabled()) {
			logger.trace("Arguments: " + Arrays.toString(args));
		}
		return doInvoke(args);
	}

	// Get the method argument values for the current request, 
	// checking the provided argument values and falling back to the configured argument resolvers.
	// The resulting array will be passed into doInvoke.
	protected Object[] getMethodArgumentValues(NativeWebRequest request, @Nullable ModelAndViewContainer mavContainer, Object... providedArgs) throws Exception {

		MethodParameter[] parameters = getMethodParameters();
		if (ObjectUtils.isEmpty(parameters)) {
			return EMPTY_ARGS;
		}

		Object[] args = new Object[parameters.length];
		for (int i = 0; i < parameters.length; i++) {
			MethodParameter parameter = parameters[i];
			parameter.initParameterNameDiscovery(this.parameterNameDiscoverer);
			args[i] = findProvidedArgument(parameter, providedArgs);
			if (args[i] != null) {
				continue;
			}
			if (!this.resolvers.supportsParameter(parameter)) {
				throw new IllegalStateException(formatArgumentError(parameter, "No suitable resolver"));
			}
			try {
				args[i] = this.resolvers.resolveArgument(parameter, mavContainer, request, this.dataBinderFactory);
			}
			catch (Exception ex) {
				// Leave stack trace for later, exception may actually be resolved and handled...
				if (logger.isDebugEnabled()) {
					String exMsg = ex.getMessage();
					if (exMsg != null && !exMsg.contains(parameter.getExecutable().toGenericString())) {
						logger.debug(formatArgumentError(parameter, exMsg));
					}
				}
				throw ex;
			}
		}
		return args;
	}

	...

	@Nullable
	protected Object doInvoke(Object... args) throws Exception {
      
		// getBridgedMethod : 우리가 작성한 Controller/Method
		ReflectionUtils.makeAccessible(getBridgedMethod());
		try {
			return getBridgedMethod().invoke(getBean(), args);
		}
		catch (IllegalArgumentException ex) {
			assertTargetBean(getBridgedMethod(), getBean(), args);
			String text = (ex.getMessage() != null ? ex.getMessage() : "Illegal argument");
			throw new IllegalStateException(formatInvokeError(text, args), ex);
		}
		catch (InvocationTargetException ex) {
			// Unwrap for HandlerExceptionResolvers ...
			Throwable targetException = ex.getTargetException();
			if (targetException instanceof RuntimeException) {
				throw (RuntimeException) targetException;
			}
			else if (targetException instanceof Error) {
				throw (Error) targetException;
			}
			else if (targetException instanceof Exception) {
				throw (Exception) targetException;
			}
			else {
				throw new IllegalStateException(formatInvokeError("Invocation failure", args), targetException);
			}
		}
	}

}
```

```java
// getArgumentResolver() 통해, resolver 들의 supportsParameter 여부 판단 -> 한개 획득
// resolveArgument
public class HandlerMethodArgumentResolverComposite implements HandlerMethodArgumentResolver {

	...   

	@Override
	@Nullable
	public Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer, NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception {

		HandlerMethodArgumentResolver resolver = getArgumentResolver(parameter);
		if (resolver == null) {
			throw new IllegalArgumentException("Unsupported parameter type [" +
					parameter.getParameterType().getName() + "]. supportsParameter should be called first.");
		}
		return resolver.resolveArgument(parameter, mavContainer, webRequest, binderFactory);
	}

	@Nullable
	private HandlerMethodArgumentResolver getArgumentResolver(MethodParameter parameter) {
		HandlerMethodArgumentResolver result = this.argumentResolverCache.get(parameter);
		if (result == null) {
			for (HandlerMethodArgumentResolver resolver : this.argumentResolvers) {
				if (resolver.supportsParameter(parameter)) {
					result = resolver;
					this.argumentResolverCache.put(parameter, result);
					break;
				}
			}
		}
		return result;
	}

	...
}
```

**e.g. RequestResponseBodyMethodProcessor**



```java
public class RequestResponseBodyMethodProcessor extends AbstractMessageConverterMethodProcessor {
   
	...

	@Override
	public boolean supportsParameter(MethodParameter parameter) {
		return parameter.hasParameterAnnotation(RequestBody.class);
	}

	...

	// HttpMessageConverter 들을 순회하며 변환
	@Override
	public Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer, NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception {
      
		...

		Object arg = readWithMessageConverters(webRequest, parameter, parameter.getNestedGenericParameterType());

		...

		return adaptArgumentIfNecessary(arg, parameter);
	}

	...
}
```

<br><br>

### 10. (Reflection) Method.invoke()


```java
ReflectionUtils.makeAccessible(getBridgedMethod());

try {
   return getBridgedMethod().invoke(getBean(), args);
}
```

(1) reflect.Method.class :: invoke() <br>
(2) DelegatingMethodAccessorImpl :: invoke() <bR>
(3) NativeMethodAccessorImpl :: invoke()   <br>

<br><br>
	
## 참고

>Reference
>1. https://medium.com/@fntldpf12/dispatcher-servlet%EC%9D%B4%EB%9E%80-624a2195d38f
> 2. https://velog.io/@seculoper235/2.-DispatcherServlet-%EC%9D%B4%EB%9E%80
> 3. https://galid1.tistory.com/525
