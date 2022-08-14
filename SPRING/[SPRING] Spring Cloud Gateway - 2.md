## [[웨비나] API Gateway를 활용한 API 개발과 비즈니스 로직 실행](https://www.youtube.com/watch?v=eXZpat-ByJQ&ab_channel=NAVERCloudPlatform%3A%EB%84%A4%EC%9D%B4%EB%B2%84%ED%81%B4%EB%9D%BC%EC%9A%B0%EB%93%9C%ED%94%8C%EB%9E%AB%ED%8F%BC)

### Gateway 고려사항

- 수많은 API 요청을 처리할 수 있는 안정적인 인프라
- Scale Out 가능 여부
- 트래픽 제어 (Rate Limit, ...)
  - Gateway 를 위한 트래픽 제어
  - Internel Service 를 위한 트래픽 제어
- 모니터링
  - 사용 현황 (호출량)
  - 응답 시간
  - 오류 현황
  - 다양한 성능 정보
- 인증 (+ 인가)
  - API Key 를 이용한 제어
  - IP ACL 를 이용한 제어
- 유연한 API 관리
  - 리소스, 메서드 관리

```text
Client    --->
Mock API  --->
                G/W --->
                          Internel
```

네이버 Gateway 의 경우 아래 Endpoint 에 대한 호출이 가능하다.
- (Serverless) Cloud Function
- Server
- LB
<br>

### 인증

- IAM
  - 사용자를 생성, (Access Key, Secret Key)관리한다.
  - 사용자별로 Access Key, Secret Key 를 통해 통신한다.
- Authorizer
  - 별도 인증, 인가 시스템이 있을 때 사용할 수 있는 방법이다.
  - Cloud Function 구조로 동작한다.
- 원한다면 인증 없이 Open 할 수 있다.

<br>

### Request / Response

필요한 정보는 다음과 같을 것이다.

- URL
- Method
- Headers
  - timestamp
  - api key
  - access key
  - hmac signature
  - ...
- Body

<br><br>

## [서울 - Spring Cloud Gateway - Spencer Gibb](https://www.youtube.com/watch?v=YyNqGMeFfjA&ab_channel=VMwareTanzu)

<img src="../images/[Spring]%20Spring%20Cloud%20Gateway%20(2)_16.png" width=70%>

<img src="../images/[Spring]%20Spring%20Cloud%20Gateway%20(2)_46.png" width=70%>
