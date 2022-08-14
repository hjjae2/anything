## Spring Cloud Gateway (SCG)


### 용어

**Route** 
- SCG의 '기본 설정 그룹' 정도로 볼 수 있을 것 같음
- ID, Dest URI, 일련의 Preciates, 일련의 Filters 등이 정의<br>
  > 예를 들어, 하나의 route는 predicate 가 true 일 때 매치

**Predicate**
- Java8 의 Predicate
- Input-Type : ServerWebExchange
- HTTP request(header, parameter 등)로부터 해당 route를 매치시킬 것인지 아닌지 등을 판단하기 위해 사용된다.

**Filter** 
- (SpringFramework GatewayFilter) (before or after sending the downstream request)request, response 를 조작할 수 있음

<br>

### 동작 원리

> 어떻게 동작하는지에 대한 그림은 [여기](https://cloud.spring.io/spring-cloud-gateway/reference/html/#gateway-how-it-works)를 참고하자.

위의 그림을 간단하게 하면 아래와 같다.

`1. Client ->  2. Gateway Handler Mapping -> 3. Gateway Web Handler -> 4. Filter(s)/Proxy Filter -> 5. 각각의 Proxied Service`

<br>

**GatewayHandlerMapping** 
- determines that a reuqest mathces a route, and then it is sent to the `GatewayWebHandler`

**GatewayWebHandler** 
- runs the request through a filter chain that is specific to the request.

**Filter(s)/ProxyFilter** 
1. All "pre" filter logic is executed 
2. Then the proxy request is made
3. Then the "post" filter logic is run.

<br>

### Two ways of Configuring (Route Predicate Factories / Gateway Filter Factories)

1. Shortcuts
2. Fully expanded arguments

>\* 실제 구성 예시 코드는 [여기](https://cloud.spring.io/spring-cloud-gateway/reference/html/#configuring-route-predicate-factories-and-gateway-filter-factories)를 참조하자.

<br>

### Route Preddicate Factories (<-> Gateway Filter Factories)

>\* 이미 내장된(정의된) `Route Predicate Factories` 와 구체적인 예시는 [여기](https://cloud.spring.io/spring-cloud-gateway/reference/html/#gateway-request-predicates-factories)에서 확인해보자.

We can combine multiple `route predicate factories` with logical `and` statements.

<br>

내장된(정의된) Route Predicate Factories 에 대해서 대략적으로 살펴보면 아래와 같은 것들이 있다.

- Beefore : 특정 시간 이전인지 체크
- After : 특정 시간 이후인지 체크
- Betwwen : 특정 시간 사이인지 체크
- Cookie : Cookie의 name,value(with regexp) 체크
- Header : Header name, value(with regexp) 체크
- [Host](https://cloud.spring.io/spring-cloud-gateway/reference/html/#the-host-route-predicate-factory) : Host 체크 (host header)
- Method : HTTP Method(GET, POST, ...) 체크
- Path : (API) Path 체크
- Query : (API) Query 체크 (name, value)
- RemoteAddr : (request's) IP 주소(Ipv4, IPv6) 체크 (CIDR-notation)
- Weight Route : 각 그룹별 weight 를 체크/맞춰준다.

<br>

### `GatewayFilter` Factories (<-> Route Predicate Factories)

>\* 이미 내장된(정의된) `GatewayFilter Factories` 와 구체적인 예시는 [여기](https://cloud.spring.io/spring-cloud-gateway/reference/html/#gatewayfilter-factories)에서 확인해보자.

This allows **'the modification' of the incoming HTTP request/outgoing HTTP response** in some manner.

This is scoped to a particular 'route'.

<br>

내장된(정의된) Route Predicate Factories 에 대해서 대략적으로 살펴보면 아래와 같은 것들이 있다.

- AddRequestHeader : Header 에 name, value 추가 (to the downstream request's header)
- AddRequestParameter : Parameter 에 name, value 추가 (**to the request's query string**)
- AddResponseHader : Header 에 name, value 추가 (to the downstream response's header)
- DedupeResponseHeader : name, strategy 값을 받고, Header name 의 값에서 strategy 전략을 통해 중복된 값을 제거
- [CircuitBreaker](https://cloud.spring.io/spring-cloud-gateway/reference/html/#spring-cloud-circuitbreaker-filter-factory) : Uses Spring Cloud CircuitBreaker APIs to wrap Gateway routes in a circuit breaker
- [FallbackHeaders](https://cloud.spring.io/spring-cloud-gateway/reference/html/#fallback-headers) : When Spring Cloud CircuitBreaker's execution exception happened, forward to a fallbackUri
- MapRequestHeader : fromHeader, toHeader 받고, 새로운 헤더를 추가(fromHeader -> toHeader)
- PrefixPath : (API) path 에 prefix 추가
- PreserveHostHeader : This filter sets a request attribute that the routing filter inspects to determine if the original host header should be sent, rather than the host header determined by the HTTP client
- [RequestRateLimiter](https://cloud.spring.io/spring-cloud-gateway/reference/html/#the-requestratelimiter-gatewayfilter-factory) : ReateLimiter 사용 -> 요청 수 제한 (HTTP 429 - Too Many Requests) (with KeyResolver)
- RedirectTo : status, uri 받고, redirect
- RemoveRequestHeader : name 받고, Header 의 name 제거
- RemoveResponseHeader : name 받고, Header 의 name 제거
- RemoveRequestParameter : name 받고, (API) **query parameter** 제거
- [RewritePath](https://cloud.spring.io/spring-cloud-gateway/reference/html/#the-rewritepath-gatewayfilter-factory) : (request)Path 에 대해 rewrite (자주 사용)
- [RewriteLocationResponseHeader](https://cloud.spring.io/spring-cloud-gateway/reference/html/#rewritelocationresponseheader-gatewayfilter-factory) : Modifies the value of the `Location (of response header)` (usually to get rid of backend-specific details)
- RewriteResponseHeader : Header 정보 수정 (마스킹, 값 조작 등)
- SaveSession : before forwarding the call downstream, forces a `WebSession::save` operation
- [SecureHeaders](https://cloud.spring.io/spring-cloud-gateway/reference/html/#the-secureheaders-gatewayfilter-factory) : response header 에 여러 정보를 추가
- SetPath : (API) Path 조작 (vs RewritePath ?)
- SetRequestHeader : Request Header 에 name, value set (vs AddRequestHeader 와의 차이는 set, add?)
- SetResponseHeader : Response Header 에 name, value set
- SetStatus : HTTP response status 설정
- StripPrefix : To strip from the request path(?)
- [Retry](https://cloud.spring.io/spring-cloud-gateway/reference/html/#the-secureheaders-gatewayfilter-factory) : 규칙에 따라 retry (자주 사용)
- RequestSize : reuqest size 로 제한 여부
- SetRequestHost : header host 를 replace
- ModifyRequestBody : request body 를 수정
- ModifyResponseBody : response body 를 수정
- Default : To add, apply a filter to all routes, we can use `default-filters`

<br>

### Global Filters

It has the same signature as `GatewayFilter`.

It is special filter that are conditionally applied to all routes. (vs Default filter (?))

...