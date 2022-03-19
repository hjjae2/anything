
## 동작 순서 : ApplicationEventPublisher.publishEvent()

### 간단 요약

1. `AbstractApplicationContext.publishEvent()` 호출
   1. ApplicationEvent 타입 확인 & Wrapping (PayloadApplicationEvent)
   2. Multicast to listeners
2. `ApplicationEventMulticaster.multicastEvent()` 호출 
   1. (Default) `SimpleApplicationEventMulticaster`
      1. Listener loop 돌며 invoke()


<br><br>

```java
public void myMethod() {
    ...

    applicationEventPublisher.publishEvent(myObject);
}
```

<br>

### AbstractApplicationContext

```java
public abstract class AbstractApplicationContext extends DefaultResourceLoader implements ConfigurableApplicationContext {

    ...


	/**
	 * Publish the given event to all listeners.
	 * Note: Listeners get initialized after the MessageSource, 
     * to be able to access it within listener implementations. 
     *
     * Thus, MessageSource implementations cannot publish events.
     *
	 * @param event the event to publish (may be an ApplicationEvent
	 * or a payload object to be turned into a PayloadApplicationEvent)
	 */
	@Override
	public void publishEvent(Object event) {
		publishEvent(event, null);
	}


	/**
	 * Publish the given event to all listeners.
     *
	 * @param event the event to publish 
     * (may be an ApplicationEvent or a payload object to be turned into a PayloadApplicationEvent)
	 * @param eventType the resolved event type, if known
	 * @since 4.2
	 */
	protected void publishEvent(Object event, @Nullable ResolvableType eventType) {
		Assert.notNull(event, "Event must not be null");

		// Decorate event as an ApplicationEvent if necessary
		ApplicationEvent applicationEvent;
		if (event instanceof ApplicationEvent) {
			applicationEvent = (ApplicationEvent) event;
		}
		else {
			applicationEvent = new PayloadApplicationEvent<>(this, event);
			if (eventType == null) {
				eventType = ((PayloadApplicationEvent<?>) applicationEvent).getResolvableType();
			}
		}

		// Multicast right now if possible - or lazily once the multicaster is initialized
		if (this.earlyApplicationEvents != null) {
			this.earlyApplicationEvents.add(applicationEvent);
		}
		else {
			getApplicationEventMulticaster().multicastEvent(applicationEvent, eventType);
            // 참고 : ApplicationEventMulticaster, SimpleApplicationEventMulticaster
		}

		// Publish event via parent context as well...
		if (this.parent != null) {
			if (this.parent instanceof AbstractApplicationContext) {
				((AbstractApplicationContext) this.parent).publishEvent(event, eventType);
			}
			else {
				this.parent.publishEvent(event);
			}
		}
	}
}
```

**1. Event(Object) 타입 변환 : `ApplicationEvent`, `PayloadApplicationEvent`**

- PayloadApplicationEvent : Wrapper 클래스

**2. 이벤트 멀티캐스팅 : `ApplicationEventMulticaster.multicastEvent`**

- 기본 : `SimpleApplicationEventMulticaster.multicastEvent();`


<br><br>

### (ApplicationEventMulticaster) SimpleApplicationEventMulticaster
```java
public class SimpleApplicationEventMulticaster extends AbstractApplicationEventMulticaster {

    ...

	@Override
	public void multicastEvent(final ApplicationEvent event, @Nullable ResolvableType eventType) {
        // ResolvableType => PayloadApplicationEvent (Object 의 Wrapper 클래스)
		ResolvableType type = (eventType != null ? eventType : resolveDefaultEventType(event));
		Executor executor = getTaskExecutor();
		for (ApplicationListener<?> listener : getApplicationListeners(event, type)) {
			if (executor != null) {
				executor.execute(() -> invokeListener(listener, event));
			}
			else {
				invokeListener(listener, event);
			}
		}
	}

    protected void invokeListener(ApplicationListener<?> listener, ApplicationEvent event) {
		ErrorHandler errorHandler = getErrorHandler();
		if (errorHandler != null) {
			try {
				doInvokeListener(listener, event);
			}
			catch (Throwable err) {
				errorHandler.handleError(err);
			}
		}
		else {
			doInvokeListener(listener, event);
		}
	}

    private void doInvokeListener(ApplicationListener listener, ApplicationEvent event) {
		try {
            // ApplicationListenerMethodAdapter
			listener.onApplicationEvent(event);
		}
		catch (ClassCastException ex) {
			String msg = ex.getMessage();
			if (msg == null || matchesClassCastMessage(msg, event.getClass()) ||
					(event instanceof PayloadApplicationEvent &&
							matchesClassCastMessage(msg, ((PayloadApplicationEvent) event).getPayload().getClass()))) {
				// Possibly a lambda-defined listener which we could not resolve the generic event type for
				// -> let's suppress the exception.
				Log loggerToUse = this.lazyLogger;
				if (loggerToUse == null) {
					loggerToUse = LogFactory.getLog(getClass());
					this.lazyLogger = loggerToUse;
				}
				if (loggerToUse.isTraceEnabled()) {
					loggerToUse.trace("Non-matching event type for listener: " + listener, ex);
				}
			}
			else {
				throw ex;
			}
		}
	}

    ...
}
```

**3. ApplicationListenerMethodAdapter onApplicationEvent(Event) 호출**

- 결국 ApplicationListner 의 onApplicationEvent 를 호출하는 것

<br><br>

### ApplicationListenerMethodAdapter

```java
public class ApplicationListenerMethodAdapter implements GenericApplicationListener {

    ...

    @Override
	public void onApplicationEvent(ApplicationEvent event) {
		processEvent(event);
	}

    public void processEvent(ApplicationEvent event) {
		Object[] args = resolveArguments(event);    // --> Event Object (e.g. Member member)
		if (shouldHandle(event, args)) {
			Object result = doInvoke(args);
            // 이벤트 핸들러(메서드)의 결과 값에 따라 `handleResult` 가 실행된다.
            // 결과 값이 있으면, 또 publish (?)
			if (result != null) {
				handleResult(result);
			}
			else {
				logger.trace("No result object given - no result to handle");
			}
		}
	}

    private boolean shouldHandle(ApplicationEvent event, @Nullable Object[] args) {
		if (args == null) {
			return false;
		}
		String condition = getCondition();
		if (StringUtils.hasText(condition)) {
			Assert.notNull(this.evaluator, "EventExpressionEvaluator must not be null");
			return this.evaluator.condition(
					condition, event, this.targetMethod, this.methodKey, args, this.applicationContext);
		}
		return true;
	}

    @Nullable
	protected Object doInvoke(Object... args) {
		Object bean = getTargetBean();  // EventListener Bean (e.g. SessionMemberLoginEventHandler)
		// Detect package-protected NullBean instance through equals(null) check
		if (bean.equals(null)) {
			return null;
		}

		ReflectionUtils.makeAccessible(this.method);
		try {
			return this.method.invoke(bean, args);  // ** 우리가 작성한 핸들러(메서드)가 실행된다. **
		}
		catch (IllegalArgumentException ex) {
			assertTargetBean(this.method, bean, args);
			throw new IllegalStateException(getInvocationErrorMessage(bean, ex.getMessage(), args), ex);
		}
		catch (IllegalAccessException ex) {
			throw new IllegalStateException(getInvocationErrorMessage(bean, ex.getMessage(), args), ex);
		}
		catch (InvocationTargetException ex) {
			// Throw underlying exception
			Throwable targetException = ex.getTargetException();
			if (targetException instanceof RuntimeException) {
				throw (RuntimeException) targetException;
			}
			else {
				String msg = getInvocationErrorMessage(bean, "Failed to invoke event listener method", args);
				throw new UndeclaredThrowableException(targetException, msg);
			}
		}
	}
    ...
}
```

> **\* 실제 메서드 호출 시 Reflection 사용!**

<br><br>

### !! Listner 가 등록되기 전에 event publishing 되는 경우

1. 일단 earlyApplicationEvents 저장한다. 
   1. `private Set<ApplicationEvent> earlyApplicationEvents;`
   2. `earlyApplicationEvents.add(Event);`
2. listener 가 등록될 때, 저장되어 있던 이벤트들을 multicast 하고 null 로 변경한다.


```java

// this.earlyApplicationEvents 가 null 인 경우 -> listener 가 등록되었다는 의미
if (this.earlyApplicationEvents != null) {
    this.earlyApplicationEvents.add(applicationEvent);
}
else {
    getApplicationEventMulticaster().multicastEvent(applicationEvent, eventType);
}
```

```java
protected void registerListeners() {
    // Register statically specified listeners first.
    for (ApplicationListener<?> listener : getApplicationListeners()) {
        getApplicationEventMulticaster().addApplicationListener(listener);
    }

    // Do not initialize FactoryBeans here: We need to leave all regular beans
    // uninitialized to let post-processors apply to them!
    String[] listenerBeanNames = getBeanNamesForType(ApplicationListener.class, true, false);
    for (String listenerBeanName : listenerBeanNames) {
        getApplicationEventMulticaster().addApplicationListenerBean(listenerBeanName);
    }

    // Publish early application events now that we finally have a multicaster...
    // 1. this.earlyApplicationEvents null 처리
    // 2. multicastEvent 호출
    Set<ApplicationEvent> earlyEventsToProcess = this.earlyApplicationEvents;
    this.earlyApplicationEvents = null;
    if (!CollectionUtils.isEmpty(earlyEventsToProcess)) {
        for (ApplicationEvent earlyEvent : earlyEventsToProcess) {
            getApplicationEventMulticaster().multicastEvent(earlyEvent);
        }
    }
}
```
