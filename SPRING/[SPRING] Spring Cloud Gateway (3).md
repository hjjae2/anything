## [`spring-cloud-starter-gateway`](https://github.com/spring-cloud/spring-cloud-gateway)

### Dependency

- spring-cloud-starter
- spring-boot-starter-webflux
  - netty
  - ...
- spring-cloud-starter-loadbalancer 
  - (optional)
- spring-cloud-gateway-server
  - 실질적으로 이 모듈에 Gateway와 관련된 클래스(코드)가 있다고 보면 될 것 같다.

<br>

## [`spring-cloud-gateway-server`](https://github.com/spring-cloud/spring-cloud-gateway/tree/main/spring-cloud-gateway-server)

### [Dependency](https://github.com/spring-cloud/spring-cloud-gateway/blob/main/spring-cloud-gateway-server/pom.xml)

> 기본적인 Dependency 가 모두 들어있다. (다만, optional 활성 여부를 꼭 확인할 것)

- spring-boot-starter
- spring-boot-starter-validation
- io.projectreactor.addons:reactor-extra
- spring-boot-starter-oauth2-client (optional)
- spring-boot-starter-actuator (optional)
- ...

### Flow


```text
HttpServerOperations (reactor-netty-http)
    --->
        HttpWebHandlerAdapter(spring-web)
                          ---> (WebHandler.handle(), DispatcherHandler.handle())
                                RoutePredicateHandlerMapping(spring-cloud-gateway-server)
                                                      --->  
                                                          FilteringWebHandler.handle()
```

<br>

### RoutePredicateHandlerMapping (AbstractHandlerMapping, HandlerMapping)


**DispatcherHandler (WebHandler)**

```java
public class DispatcherHandler implements WebHandler, PreFlightRequestHandler, ApplicationContextAware {

  ... 

	@Override
	public Mono<Void> handle(ServerWebExchange exchange) {
		if (this.handlerMappings == null) {
			return createNotFoundError();
		}
		if (CorsUtils.isPreFlightRequest(exchange.getRequest())) {
			return handlePreFlight(exchange);
		}
		return Flux.fromIterable(this.handlerMappings)
				.concatMap(mapping -> mapping.getHandler(exchange)) // 여기 : AbstractHandlerMapping.getHandler() 로 이어진다.
				.next()
				.switchIfEmpty(createNotFoundError())
				.flatMap(handler -> invokeHandler(exchange, handler))
				.flatMap(result -> handleResult(exchange, result));
	}

  ...

}
```

<br>

**AbstractHandlerMapping**

```java
public abstract class AbstractHandlerMapping extends ApplicationObjectSupport implements HandlerMapping, Ordered, BeanNameAware {

  @Override
	public Mono<Object> getHandler(ServerWebExchange exchange) {
		return getHandlerInternal(exchange).map(handler -> {      // 여기 : RoutePredicateHandlerMapping.getHandlerInternal() 로 이어진다.
			if (logger.isDebugEnabled()) {
				logger.debug(exchange.getLogPrefix() + "Mapped to " + handler);
			}
			ServerHttpRequest request = exchange.getRequest();
			if (hasCorsConfigurationSource(handler) || CorsUtils.isPreFlightRequest(request)) {
				CorsConfiguration config = (this.corsConfigurationSource != null ?
						this.corsConfigurationSource.getCorsConfiguration(exchange) : null);
				CorsConfiguration handlerConfig = getCorsConfiguration(handler, exchange);
				config = (config != null ? config.combine(handlerConfig) : handlerConfig);
				if (config != null) {
					config.validateAllowCredentials();
				}
				if (!this.corsProcessor.process(config, exchange) || CorsUtils.isPreFlightRequest(request)) {
					return NO_OP_HANDLER;
				}
			}
			return handler;
		});
	}

}
```

```java
public class RoutePredicateHandlerMapping extends AbstractHandlerMapping {

	private final FilteringWebHandler webHandler;

  private final RouteLocator routeLocator;

  ...

  @Override
	protected Mono<?> getHandlerInternal(ServerWebExchange exchange) {
		// don't handle requests on management port if set and different than server port
		if (this.managementPortType == DIFFERENT && this.managementPort != null
				&& exchange.getRequest().getURI().getPort() == this.managementPort) {
			return Mono.empty();
		}
		exchange.getAttributes().put(GATEWAY_HANDLER_MAPPER_ATTR, getSimpleName());

		return lookupRoute(exchange)
				// .log("route-predicate-handler-mapping", Level.FINER) //name this
				.flatMap((Function<Route, Mono<?>>) r -> {
					exchange.getAttributes().remove(GATEWAY_PREDICATE_ROUTE_ATTR);
					if (logger.isDebugEnabled()) {
						logger.debug("Mapping [" + getExchangeDesc(exchange) + "] to " + r);
					}

					exchange.getAttributes().put(GATEWAY_ROUTE_ATTR, r);
					return Mono.just(webHandler);     // ← 여기 : FilteringWebHandler 를 반환한다.
				}).switchIfEmpty(Mono.empty().then(Mono.fromRunnable(() -> {
					exchange.getAttributes().remove(GATEWAY_PREDICATE_ROUTE_ATTR);
					if (logger.isTraceEnabled()) {
						logger.trace("No RouteDefinition found for [" + getExchangeDesc(exchange) + "]");
					}
				})));
	}

  ...

	protected Mono<Route> lookupRoute(ServerWebExchange exchange) {
		return this.routeLocator.getRoutes()
				// individually filter routes so that filterWhen error delaying is not a
				// problem
				.concatMap(route -> Mono.just(route).filterWhen(r -> {
					// add the current route we are testing
					exchange.getAttributes().put(GATEWAY_PREDICATE_ROUTE_ATTR, r.getId());
					return r.getPredicate().apply(exchange);
				})
						// instead of immediately stopping main flux due to error, log and
						// swallow it
						.doOnError(e -> logger.error("Error applying predicate for route: " + route.getId(), e))
						.onErrorResume(e -> Mono.empty()))
				// .defaultIfEmpty() put a static Route not found
				// or .switchIfEmpty()
				// .switchIfEmpty(Mono.<Route>empty().log("noroute"))
				.next()
				// TODO: error handling
				.map(route -> {
					if (logger.isDebugEnabled()) {
						logger.debug("Route matched: " + route.getId());
					}
					validateRoute(route, exchange);
					return route;
				});

		/*
		 * TODO: trace logging if (logger.isTraceEnabled()) {
		 * logger.trace("RouteDefinition did not match: " + routeDefinition.getId()); }
		 */
	}
```

```log
2022-07-17 20:01:37.968 DEBUG 91322 --- [ctor-http-nio-2] o.s.c.g.h.RoutePredicateHandlerMapping   : Route matched: 4162e05b-be86-4b3b-a2c7-bfcc4d2c89b4
2022-07-17 20:01:37.968 DEBUG 91322 --- [ctor-http-nio-2] o.s.c.g.h.RoutePredicateHandlerMapping   : Mapping [Exchange: GET http://localhost:8080/api/samples] to Route{id='4162e05b-be86-4b3b-a2c7-bfcc4d2c89b4', uri=http://localhost:8081, order=0, predicate=PredicateSpec$$Lambda$529/0x0000000800510c40, gatewayFilters=[[ModifyResponseBody New content type = [null], In class = Object, Out class = ResponseFormatFilter.Response]], metadata={}}
2022-07-17 20:01:37.968 DEBUG 91322 --- [ctor-http-nio-2] o.s.c.g.h.RoutePredicateHandlerMapping   : [d9c36f28-2] Mapped to org.springframework.cloud.gateway.handler.FilteringWebHandler@2937f9e0
```

<br>

**FilteringWebHandler**

```java
public class FilteringWebHandler implements WebHandler {

	protected static final Log logger = LogFactory.getLog(FilteringWebHandler.class);

	private final List<GatewayFilter> globalFilters;

	...

	@Override
	public Mono<Void> handle(ServerWebExchange exchange) {
		Route route = exchange.getRequiredAttribute(GATEWAY_ROUTE_ATTR);
		List<GatewayFilter> gatewayFilters = route.getFilters();

		List<GatewayFilter> combined = new ArrayList<>(this.globalFilters);
		combined.addAll(gatewayFilters);
		// TODO: needed or cached?
		AnnotationAwareOrderComparator.sort(combined);

		if (logger.isDebugEnabled()) {
			logger.debug("Sorted gatewayFilterFactories: " + combined);
		}

		return new DefaultGatewayFilterChain(combined).filter(exchange);    // 여기 : Filter Chain 에 넘긴다.
	}

  ...
}
```

<br>

**DefaultGatewayFilterChain**

```java
public class FilteringWebHandler implements WebHandler {

  ...
  
	private static class DefaultGatewayFilterChain implements GatewayFilterChain {

		private final int index;

		private final List<GatewayFilter> filters;

		...

		@Override
		public Mono<Void> filter(ServerWebExchange exchange) {
			return Mono.defer(() -> {
				if (this.index < filters.size()) {
					GatewayFilter filter = filters.get(this.index);
					DefaultGatewayFilterChain chain = new DefaultGatewayFilterChain(this, this.index + 1);
					return filter.filter(exchange, chain);  // 여기 : 우리가 등록한 filter(혹은 기본적으로 등록된 filter) 들이 동작한다.
				}
				else {
					return Mono.empty(); // complete
				}
			});
		}

	}
```

아래 로그는 FilteringWebHandler 로그이다.

> 등록된 Filter 들을 확인할 수 있다. 
```log
2022-07-17 20:01:37.968 DEBUG 91322 --- [ctor-http-nio-2] o.s.c.g.handler.FilteringWebHandler      : Sorted gatewayFilterFactories: [[GatewayFilterAdapter{delegate=org.springframework.cloud.gateway.filter.RemoveCachedBodyFilter@5707f613}, order = -2147483648], [GatewayFilterAdapter{delegate=org.springframework.cloud.gateway.filter.AdaptCachedBodyGlobalFilter@526e8108}, order = -2147482648], [ModifyResponseBody New content type = [null], In class = Object, Out class = ResponseFormatFilter.Response], [GatewayFilterAdapter{delegate=org.springframework.cloud.gateway.filter.NettyWriteResponseFilter@11787b64}, order = -1], [GatewayFilterAdapter{delegate=org.springframework.cloud.gateway.filter.ForwardPathFilter@319642db}, order = 0], [GatewayFilterAdapter{delegate=org.springframework.cloud.gateway.filter.GatewayMetricsFilter@35bfa1bb}, order = 0], [GatewayFilterAdapter{delegate=org.springframework.cloud.gateway.filter.RouteToRequestUrlFilter@77b3752b}, order = 10000], [GatewayFilterAdapter{delegate=org.springframework.cloud.gateway.config.GatewayNoLoadBalancerClientAutoConfiguration$NoLoadBalancerClientFilter@6b321262}, order = 10150], [GatewayFilterAdapter{delegate=org.springframework.cloud.gateway.filter.WebsocketRoutingFilter@59498d94}, order = 2147483646], GatewayFilterAdapter{delegate=com.baemin.openapi.filter.global.SampleCustomGlobalGatewayFilter@713a35c5}, [GatewayFilterAdapter{delegate=org.springframework.cloud.gateway.filter.NettyRoutingFilter@62aeddc8}, order = 2147483647], [GatewayFilterAdapter{delegate=org.springframework.cloud.gateway.filter.ForwardRoutingFilter@6367a688}, order = 2147483647]]

```

![](../images/[SPRING]%20Spring%20Cloud%20Gateway%20(3)_50.png)

<br>

아래는 하나의 Request에 대한 전체 로그이다.

```... http-nio-2] r.n.http.server.HttpServerOperations     : [d9c36f28, L:/0:0:0:0:0:0:0:1:8080 - R:/0:0:0:0:0:0:0:1:53838] Increasing pending responses, now 1
... [ctor-http-nio-2] reactor.netty.http.server.HttpServer     : [d9c36f28-2, L:/0:0:0:0:0:0:0:1:8080 - R:/0:0:0:0:0:0:0:1:53838] Handler is being applied: org.springframework.http.server.reactive.ReactorHttpHandlerAdapter@687105a6
... [ctor-http-nio-2] o.s.w.s.adapter.HttpWebHandlerAdapter    : [d9c36f28-2] HTTP GET "/api/samples"
... [ctor-http-nio-2] o.s.c.g.h.RoutePredicateHandlerMapping   : Route matched: 4162e05b-be86-4b3b-a2c7-bfcc4d2c89b4
... [ctor-http-nio-2] o.s.c.g.h.RoutePredicateHandlerMapping   : Mapping [Exchange: GET http://localhost:8080/api/samples] to Route{id='4162e05b-be86-4b3b-a2c7-bfcc4d2c89b4', uri=http://localhost:8081, order=0, predicate=PredicateSpec$$Lambda$529/0x0000000800510c40, gatewayFilters=[[ModifyResponseBody New content type = [null], In class = Object, Out class = ResponseFormatFilter.Response]], metadata={}}
... [ctor-http-nio-2] o.s.c.g.h.RoutePredicateHandlerMapping   : [d9c36f28-2] Mapped to org.springframework.cloud.gateway.handler.FilteringWebHandler@2937f9e0
... [ctor-http-nio-2] o.s.c.g.handler.FilteringWebHandler      : Sorted gatewayFilterFactories: [[GatewayFilterAdapter{delegate=org.springframework.cloud.gateway.filter.RemoveCachedBodyFilter@5707f613}, order = -2147483648], [GatewayFilterAdapter{delegate=org.springframework.cloud.gateway.filter.AdaptCachedBodyGlobalFilter@526e8108}, order = -2147482648], [ModifyResponseBody New content type = [null], In class = Object, Out class = ResponseFormatFilter.Response], [GatewayFilterAdapter{delegate=org.springframework.cloud.gateway.filter.NettyWriteResponseFilter@11787b64}, order = -1], [GatewayFilterAdapter{delegate=org.springframework.cloud.gateway.filter.ForwardPathFilter@319642db}, order = 0], [GatewayFilterAdapter{delegate=org.springframework.cloud.gateway.filter.GatewayMetricsFilter@35bfa1bb}, order = 0], [GatewayFilterAdapter{delegate=org.springframework.cloud.gateway.filter.RouteToRequestUrlFilter@77b3752b}, order = 10000], [GatewayFilterAdapter{delegate=org.springframework.cloud.gateway.config.GatewayNoLoadBalancerClientAutoConfiguration$NoLoadBalancerClientFilter@6b321262}, order = 10150], [GatewayFilterAdapter{delegate=org.springframework.cloud.gateway.filter.WebsocketRoutingFilter@59498d94}, order = 2147483646], GatewayFilterAdapter{delegate=com.baemin.openapi.filter.global.SampleCustomGlobalGatewayFilter@713a35c5}, [GatewayFilterAdapter{delegate=org.springframework.cloud.gateway.filter.NettyRoutingFilter@62aeddc8}, order = 2147483647], [GatewayFilterAdapter{delegate=org.springframework.cloud.gateway.filter.ForwardRoutingFilter@6367a688}, order = 2147483647]]
... [ctor-http-nio-2] r.n.resources.PooledConnectionProvider   : [d27fde7a, L:/127.0.0.1:53841 - R:localhost/127.0.0.1:8081] Channel acquired, now: 1 active connections, 0 inactive connections and 0 pending acquire requests.
... [ctor-http-nio-2] r.netty.http.client.HttpClientConnect    : [d27fde7a-2, L:/127.0.0.1:53841 - R:localhost/127.0.0.1:8081] Handler is being applied: {uri=http://localhost:8081/api/samples, method=GET}
... [ctor-http-nio-2] r.n.r.DefaultPooledConnectionProvider    : [d27fde7a-2, L:/127.0.0.1:53841 - R:localhost/127.0.0.1:8081] onStateChange(GET{uri=/api/samples, connection=PooledConnection{channel=[id: 0xd27fde7a, L:/127.0.0.1:53841 - R:localhost/127.0.0.1:8081]}}, [request_prepared])
... [ctor-http-nio-2] reactor.netty.channel.FluxReceive        : [d9c36f28-2, L:/0:0:0:0:0:0:0:1:8080 - R:/0:0:0:0:0:0:0:1:53838] FluxReceive{pending=0, cancelled=false, inboundDone=false, inboundError=null}: subscribing inbound receiver
... [ctor-http-nio-2] r.n.r.DefaultPooledConnectionProvider    : [d27fde7a-2, L:/127.0.0.1:53841 - R:localhost/127.0.0.1:8081] onStateChange(GET{uri=/api/samples, connection=PooledConnection{channel=[id: 0xd27fde7a, L:/127.0.0.1:53841 - R:localhost/127.0.0.1:8081]}}, [request_sent])
... [ctor-http-nio-2] r.n.http.client.HttpClientOperations     : [d27fde7a-2, L:/127.0.0.1:53841 - R:localhost/127.0.0.1:8081] Received response (auto-read:false) : [Content-Type=application/json, Transfer-Encoding=chunked, Date=Sun, 17 Jul 2022 11:01:37 GMT]
... [ctor-http-nio-2] r.n.r.DefaultPooledConnectionProvider    : [d27fde7a-2, L:/127.0.0.1:53841 - R:localhost/127.0.0.1:8081] onStateChange(GET{uri=/api/samples, connection=PooledConnection{channel=[id: 0xd27fde7a, L:/127.0.0.1:53841 - R:localhost/127.0.0.1:8081]}}, [response_received])
... [ctor-http-nio-2] reactor.netty.channel.FluxReceive        : [d27fde7a-2, L:/127.0.0.1:53841 - R:localhost/127.0.0.1:8081] FluxReceive{pending=0, cancelled=false, inboundDone=false, inboundError=null}: subscribing inbound receiver
... [ctor-http-nio-2] r.n.http.client.HttpClientOperations     : [d27fde7a-2, L:/127.0.0.1:53841 - R:localhost/127.0.0.1:8081] Received last HTTP packet
... [ctor-http-nio-2] o.s.http.codec.json.Jackson2JsonDecoder  : Decoded [{timestamp=0, errorCode=, statusCode=200, statusMessage=OK, data=[{name=, age=12}, {name=길동 (truncated)...]
... [ctor-http-nio-2] o.s.http.codec.json.Jackson2JsonEncoder  : Encoding [Response(data={timestamp=0, errorCode=, statusCode=200, statusMessage=OK, data=[{name=현재, age= (truncated)...]
... [ctor-http-nio-2] r.n.http.server.HttpServerOperations     : [d9c36f28-2, L:/0:0:0:0:0:0:0:1:8080 - R:/0:0:0:0:0:0:0:1:53838] Decreasing pending responses, now 0
... [ctor-http-nio-2] r.n.http.server.HttpServerOperations     : [d9c36f28-2, L:/0:0:0:0:0:0:0:1:8080 - R:/0:0:0:0:0:0:0:1:53838] Last HTTP packet was sent, terminating the channel
... [ctor-http-nio-2] o.s.w.s.adapter.HttpWebHandlerAdapter    : [d9c36f28-2] Completed 200 OK
... [ctor-http-nio-2] r.n.http.server.HttpServerOperations     : [d9c36f28-2, L:/0:0:0:0:0:0:0:1:8080 - R:/0:0:0:0:0:0:0:1:53838] Last HTTP response frame
... [ctor-http-nio-2] r.n.r.DefaultPooledConnectionProvider    : [d27fde7a, L:/127.0.0.1:53841 - R:localhost/127.0.0.1:8081] onStateChange(GET{uri=/api/samples, connection=PooledConnection{channel=[id: 0xd27fde7a, L:/127.0.0.1:53841 - R:localhost/127.0.0.1:8081]}}, [response_completed])
... [ctor-http-nio-2] r.n.r.DefaultPooledConnectionProvider    : [d27fde7a, L:/127.0.0.1:53841 - R:localhost/127.0.0.1:8081] onStateChange(GET{uri=/api/samples, connection=PooledConnection{channel=[id: 0xd27fde7a, L:/127.0.0.1:53841 - R:localhost/127.0.0.1:8081]}}, [disconnecting])
... [ctor-http-nio-2] r.n.r.DefaultPooledConnectionProvider    : [d27fde7a, L:/127.0.0.1:53841 - R:localhost/127.0.0.1:8081] Releasing channel
... [ctor-http-nio-2] r.n.resources.PooledConnectionProvider   : [d27fde7a, L:/127.0.0.1:53841 - R:localhost/127.0.0.1:8081] Channel cleaned, now: 0 active connections, 1 inactive connections and 0 pending acquire requests.
... [ctor-http-nio-2] r.n.r.DefaultPooledConnectionProvider    : [d27fde7a, L:/127.0.0.1:53841 ! R:localhost/127.0.0.1:8081] onStateChange(PooledConnection{channel=[id: 0xd27fde7a, L:/127.0.0.1:53841 ! R:localhost/127.0.0.1:8081]}, [disconnecting])
```

<br>

### 참고 (L, R, -, !)

> L : Local Address <br>
> R : Remote Address <br>
> \- : active (active and so connected) <br>
> ! : not active <br>


<br>

**`AbstractChannel` 에서 확인할 수 있다.**

```java
public abstract class AbstractChannel extends DefaultAttributeMap implements Channel {

  ...

  @Override
  public String toString() {
      boolean active = isActive();
      if (strValActive == active && strVal != null) {
          return strVal;
      }

      SocketAddress remoteAddr = remoteAddress();
      SocketAddress localAddr = localAddress();
      if (remoteAddr != null) {
          StringBuilder buf = new StringBuilder(96)
              .append("[id: 0x")
              .append(id.asShortText())
              .append(", L:")
              .append(localAddr)
              .append(active? " - " : " ! ")
              .append("R:")
              .append(remoteAddr)
              .append(']');
          strVal = buf.toString();
      } else if (localAddr != null) {
          StringBuilder buf = new StringBuilder(64)
              .append("[id: 0x")
              .append(id.asShortText())
              .append(", L:")
              .append(localAddr)
              .append(']');
          strVal = buf.toString();
      } else {
          StringBuilder buf = new StringBuilder(16)
              .append("[id: 0x")
              .append(id.asShortText())
              .append(']');
          strVal = buf.toString();
      }

      strValActive = active;
      return strVal;
  }

  ...
}
```