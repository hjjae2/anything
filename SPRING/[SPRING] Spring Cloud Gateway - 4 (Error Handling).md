Gateway 에서 에러 발생 시, 핸들링할 수 있는 `WebExceptionHandler`를 구현합니다.

## 구현 내용

1. `ErrorWebExceptionHandler` 인터페이스 구현 → `GlobalErrorWebExceptionHandler`
2. `ErrorAttributes` 인터페이스 구현 → `GlobalErrorAttributes`

<br>

### 흐름

<img src="../images/[SPRING]%20Spring%20Cloud%20Gateway%20-%204%20(Error%20Handling)_59.png" width="80%">

- `ExceptionHandlingWebHandler`  가 `FilteringWebHandler` 에 Request 처리를 위임하는 방식 (참고 : `WebHttpHandlerBuilder.build()`)

<br>

### 연관 클래스

|클래스|설명 / 역할|
|-|-|
|`ExceptionHandlingWebHandler`|`WebExceptionHandler` List 포함 <br><br> exceptionHandler 에러를 핸들링한다.|
|`WebExceptionHandler`|Contract for handling exceptions during web server exchange processing. <br><br> `GlobalErrorWebExceptionHandler` 이 해당 <br><br> WebHandler 에서 Exception 발생 시, `WebExceptionHandler`(`GlobalErrorWebExceptionHandler`) 가 처리|
|`ErrorAttributes`|Provides access to error attributes which can be logged or presented to the user. <br><br> 반환할, 로깅할 Error Attributes 를 제공하는 클래스 <br><br> `GlobalErrorAttributes` 이 해당|

<br>

**예시 - ExceptionHandlingWebHandler.handle()**

```java
public Mono<Void> handle(ServerWebExchange exchange) {
	Mono<Void> completion;
	try {
		completion = super.handle(exchange);
	}
	catch (Throwable ex) {
		completion = Mono.error(ex);
	}

	for (WebExceptionHandler handler : this.exceptionHandlers) {
		completion = completion.onErrorResume(ex -> handler.handle(exchange, ex));
	}
	return completion;
}
```

<br>

**예시 - 기본 구현체 (DefaultErrorAttributes) 는 아래 에러 컨텐츠를 반환**

```json
{
    "timestamp": "2022-08-10T08:11:13.608+00:00",
    "path": "/api2/2xx",
    "status": 404,
    "error": "Not Found",
    "message": null,
    "requestId": "5c49ed8a-1"
}
```

<br><br>

### 참고
1. https://docs.spring.io/spring-framework/docs/5.1.6.RELEASE/spring-framework-reference/web-reactive.html#webflux-exception-handler
2. https://docs.spring.io/spring-boot/docs/2.1.4.RELEASE/reference/htmlsingle/#boot-features-webflux-error-handling


