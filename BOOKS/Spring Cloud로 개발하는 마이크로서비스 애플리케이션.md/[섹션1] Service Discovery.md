---
layout: post
title:  "SpringCloudGateway :: Service Discovery"
author: "leehyunjae"
tags: ["spring"]
---

> Service Discovery(Eureka) 이해해보기

> PC 가 한 대라면 같은 IP/다른 PORT 형태로 여러 애플리케이션을 구성할 수 있다. PC 가 여러 대라면, 다른 IP/같은 PORT 형태로 여러 애플리케이션을 구성할 수 있다.

Service Discovery 란, 외부에서 내부의 마이크로서비스를 찾기 위해(검색) 사용된다. Key, Value 쌍으로 서비스를 등록하고 검색할 수 있다. 또, Netflix Eureka 는 Netfix 에서 만든 오픈소스를 아파치 재단에 등록한 것이다.

Load Balancer 혹은 API Gateway 에 요청이 들어왔을 때, 먼저 Service Discovery 에 해당 서비스가 어디 위치하고 있는지 조회하는 개념이다.

<br>

기본적인 설정은 다음과 같이 할 수 있다.

**Eureka Server**

> 127.0.0.1:9000 으로 Eureka Server 를 가동한다고 가정한다.

```java
@EnableEurekaServer // @EnableEurekaServer 어노테이션을 추가한다.
@SpringBootApplication
public class ServiceDiscoveryApplication {
	public static void main(String[] args) {
		SpringApplication.run(ServiceDiscoveryApplication.class, args);
	}
}

```

```yaml
server:
  port: 9000

spring:
  application:
    name: service-discovery

eureka:
  client:
    fetch-registry: false
    register-with-eureka: false
```

<br>

**Eureka Client**

```java
@EnableDiscoveryClient // @EnableDiscoveryClient 어노테이션을 추가한다.
@SpringBootApplication
public class Demo1Application {
	public static void main(String[] args) {
		SpringApplication.run(Demo1Application.class, args);
	}
}
```

```yaml
server:
  port: 8001

spring:
  application:
    name: demo1

eureka:
  instance:
    instance-id: ${spring.application.name}:${spring.cloud.client.hostname}:${server.port}:${spring.application.instance_id:${random.value}} # instance id 의 경우 자유롭게 설정하면 된다. 여기서는 instance id 가 중복되지 않도록 하기 위해 random.value 를 추가해주었다.
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://127.0.0.1:9000/eureka # Eureka Server 쪽을 향하도록 설정한다.
```

<br>

위와 같은 형태로 Eureka 서버, 데모1, 데모2, 게이트웨이 를 가동한다고 가정하면 아래와 같이 설정된 것을 볼 수 있다.

<br>

<img src="/images/inflearn/springclodegateway/eureka_sample.png" width="100%" height="100%" alt="OSIV">