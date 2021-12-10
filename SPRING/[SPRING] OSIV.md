---
layout: post
title: "SpringBoot :: OSIV"
author: "leehyunjae"
tags: ["springboot", "java"]
---

> Spring Framework 의 OSIV 에 대해 이해해보기

<br>

스프링에서 기본적으로 영속(준영속)상태, 트랜잭션의 범위는 아래와 같다고 한다.

<img src="/images/spring/OSIV_기본.png" width="100%" height="100%" alt="OSIV">

그림에서와 같이 Intercepter, Controller, View 단에서는 준영속 상태이기 때문에 영속 상태의 이점을 누리지 못한다. 영속 상태의 이점이란, **변경 감지, 지연 로딩** 등의 기능이다. 즉, 쉽게 보면 Controller 에서는 변경 감지나 지연 로딩의 기능을 사용할 수 없는 것이다.

```java
public class OrderController {
    ...
    public String index(Long id) {
        Order order = orderService.getOrder(id);
        order.getShop(); // 지연로딩이라면 사용할 수 없다. (예외 발생)
        order.setName("new"); // 변경감지를 사용할 수 없다.
    }
}
```

위의 문제는 **1. Eager 방식, 2. Fetch Join, 3. 강제 초기화, 4. FACADE 계층 도입 5. OSIV** 등의 해결 방법이 있다고 한다. 이들 중 OSIV 를 활용한 방법을 살펴본다.

<br>

### OSIV

'Open Session In View', 영속성 컨텍스트를 View 까지 오픈한다는 의미라고 한다. 영속성 컨텍스트를 열어둔다는 것(살려둔다는 것)은 엔티티가 영속 상태로 유지됨을 의미한다. 즉 위의 Intercepter, Controller, View 에서 영속 상태의 이점(변경 감지, 지연 로딩)을 활용할 수 있음을 의미한다.

<br>

### OSIV : 요청 당 트랜잭션

요청 당 트랜잭션의 방식은 말 그대로 요청 당 트랜잭션을 할당하는 것이다. 요청이 들어오면 Servlet Filter / Intercepter 에서 트랜잭션을 시작하여 요청이 끝날 때까지 트랜잭션이 유지되는 것이다.

<img src="/images/spring/OSIV_요청당트랜잭션.png" width="100%" height="100%" alt="OSIV">

다만 이 방식은 프레젠테이션 계층(Controller, View)에서 자유롭게 엔티티를 수정할 수 있기에 위험한 방법이라고 볼 수 있다. 

> 최근에는 이 방식을 거의 사용하지 않는다고 한다.

<br>

### OSIV : Spring Framework

Spring Frmework 에서는 위의 방식을 보완한 OSIV 를 제공한다. 비즈니스 계층에서만 트랜잭션을 유지(쓰기 작업 유지)하는 것이다. 즉, 프레젠테이션 계층에서 지연 로딩과 같이 읽기 작업에 대해서는 열어두되, 변경 감지와 같은 쓰기 작업에 대해서는 제한을 거는 것이다. 

<img src="/images/spring/OSIV_스프링.png" width="100%" height="100%" alt="OSIV">

아래와 같은 방식으로 동작한다고 한다.

1. 요청이 들어오면 Servlet Filter / Intercepter 에서 영속성 컨텍스트를 생성한다. (요청 당 트랜잭션에서는 트랜잭션을 시작한다고 했다.)
2. 비즈니스 계층에서 `@Transactional` 를 통해 트랜잭션이 시작되면 1번에서 생성해둔 영속성 컨텍스트를 찾아와 트랜잭션이 시작된다.
3. 비즈니스 계층(`@Transactional`)이 끝나면 영속성 컨텍스트는 종료되지 않고 트랜잭션은 commit 된다.
4. 프레젠테이션 계층에서는 영속성 컨텍스트가 유지된다. (이때 트랜잭션은 없다.)
5. 요청이 끝날 때 영속성 컨텍스트를 종료한다. (마찬가지로 트랜잭션이 없다. flush를 호출하지 않는다.)

다만 아래와 같은 코드 형태에 대해서 주의해야 한다고 한다.

```java
public class OrderController {
    ...
    public String index(Long id) {
        Order order = orderService.getOrder(id);
        order.getShop();
        order.setName("new");

        orderService.updateLogic(); // updateLogic 은 Transactional 이 걸려있다고 가정한다. 
    }
}
```

`orderService.updateLogic()` 이 실행되면서 트랜잭션이 다시 시작되고 끝(commit)나게 된다. 이때 `order.setName("new")` 의 코드에 대해서 변경 감지가 발동하여 쿼리가 나가게 된다.

> 이러한 문제는 단순히 비즈니스 로직을 먼저 호출하는 방식으로 피하기도 한다고 한다.


<br>

### 참고

[OSIV와 Spring Framework에서의 OSIV에 대해서](https://prolog.techcourse.co.kr/posts/1736)