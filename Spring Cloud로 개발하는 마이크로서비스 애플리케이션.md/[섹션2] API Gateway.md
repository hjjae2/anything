## API Gateway

외부에 노출되는 API 이다. 

내부의 API 들은 감추고, 외부에 노출하여 Proxy/Gateway 역할(즉 단일 진입점 역할)을 한다.

Client 는 API Gateway(단일 API) 만 바라보고 작업할 수 있다.

API Gateway 는 인증, 인가, LB, Routing, Logging, CircuitBreaker 의 역할을 한다. (?)

### Netflix Rebbon

Spring Cloud 에서 MSA 간 통신이 필요하다.

- ResetTemplate : RestAPI 를 통해 통신한다. 
- Feign Client : MicroService의 이름을 등록하고, 이름으로 호출한다.

Ribbon : Client-Side Load Balancer
- 서비스 이름으로 호출
- Health checking

\* Spring Cloud Ribbon 은 Spring Boot 2.4에서 Maintenance 상태이다.

<br>

### Netflix Zuul

API Gateway library(fraemwork) 이다.

```
client <-> netflix zuul <-> msa1, msa2, ...
```

\* Spring Cloud Zuul 은 Spring Boot 2.4에서 Maintenance 상태이다.