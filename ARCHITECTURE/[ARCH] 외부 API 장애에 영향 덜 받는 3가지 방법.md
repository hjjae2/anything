> 출처: [외부 API 장애에 영향 덜 받는 3가지 방법](https://www.youtube.com/watch?v=nuRO0ZBFdKk) 

## 외부 API 장애에 영향 덜 받는 3가지 방법 (Resilience)

### 1. 타임아웃 (Timeout)

> *" 오래 대기하는 것보다 빨리 에러를 발생시키는 것이 좋다. "*

API 연동 시 타임아웃 설정은 기본이다.

타임아웃의 종류는 대표적으로 connect timeout, read timeout 이 있다.

<br>

**connect timeout**

보통 1초~5초 이내로 설정

<br>

**read timeout**

보통 1초~3초 사이로 설정

단, API/서비스 특징에 따라 시간을 다르게 설정하기도 한다.

- 결제 승인 : 30초

<br>

**커넥션 풀 타임아웃**

![](../images/[ARCH]%20외부%20API%20장애에%20영향%20덜%20받는%203가지%20방법_56.png)

<br>

### 2. 벌크헤드 (Bulkhead)

리소스를 격리시킨다.

![](../images/[ARCH]%20외부%20API%20장애에%20영향%20덜%20받는%203가지%20방법_42.png)

![](../images/[ARCH]%20외부%20API%20장애에%20영향%20덜%20받는%203가지%20방법_30.png)

![](../images/[ARCH]%20외부%20API%20장애에%20영향%20덜%20받는%203가지%20방법_32.png)

<br>

### 3. 서킷브레이커 (Circuit Breaker)

> *생략*