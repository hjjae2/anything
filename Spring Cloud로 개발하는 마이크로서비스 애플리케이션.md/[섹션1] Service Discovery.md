## Service Discovery

> Spring Cloud Netfilx Eureka

> PC 가 한 대라면, 같은 IP / 다른 PORT 로 구성할 수 있다.
> PC 가 여러 대라면, 다른 IP / 같은 PORT 로 구성할 수 있다.

Service Discovery 란? 외부에서 내부의 마이크로서비스를 찾기 위해(검색) 사용된다. Key, Value 쌍으로 서비스를 등록하고, 검색할 수 있다.

Netfix 에서 만든 오픈소스를 아파치 재단에 등록한 것, `Netflix Eureka`

Load Balancer 혹은 API Gateway 에 요청이 들어왔을 때, 먼저 Service Discovery 에 해당 서비스가 어디 위치하고 있는지 조회하는 개념이다.
