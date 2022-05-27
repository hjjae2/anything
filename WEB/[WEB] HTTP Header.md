> 커스텀 등록 헤더는 'X-' 라고 붙이곤 했었는데 RFC 6648(2012.06)에 의해 폐기


## 분류 (by Context)

**General Header**
- request/response 공통(모두) 적용
- body에 전송되는 데이터와는 관련이 없음 (예를 들면, Content-Type 등을 의미하는 건가?)

**Request Header**
- 클라이언트 정보

**Response Header**
- 서버 정보 (이름, 버전 등)

**Entity Header**
- Entity Body 정보 (Content-Length, MIME 타입 등)


> 프록시 처리 방법에 따라 분류할 수도 있다. 
> 
> End-to-end headers, Hop-by-hop headers

<br><br>

### General Header

request / response 모두에서 사용되는 공통 헤더

컨텐츠(body)에는 적용되지 않는 헤더 (컨텐츠와 관련이 없는 헤더)

**예시**

```http
Request URL: https://developer.mozilla.org/api/v1/whoami
Request Method: GET
Status Code: 200 
Remote Address: 54.230.168.85:443
Referrer Policy: strict-origin-when-cross-origin
```

- Date
- Cache-Control
- Connection

<br><br>

### Request Header

HTTP 요청 시 사용되는 헤더

**예시**
```http
accept: */*
accept-encoding: gzip, deflate, br
accept-language: ko,en-US;q=0.9,en;q=0.8
cookie: preferredlocale=ko
referer: https://developer.mozilla.org/ko/docs/Glossary/Response_header
sec-ch-ua: " Not A;Brand";v="99", "Chromium";v="99", "Google Chrome";v="99"
sec-ch-ua-mobile: ?0
sec-ch-ua-platform: "macOS"
sec-fetch-dest: empty
sec-fetch-mode: cors
sec-fetch-site: same-origin
user-agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/99.0.4844.84 Safari/537.36
```

**엄격하게 본다면, (브라우저 확인 시) Request Header 에 포함된 헤더 모두가 Request Header 는 아님에 주의**

예를 들어, `Content-Length` 는 `Entity Header` 이다.

> *"요청에 나타나는 모든 헤더가 요청 헤더인것은 아닙니다. 예를 들면, POST 요청에 나타나는 Content-Length는 실제로 요청 메시지 바디의 크기를 참조하는 entity header입니다. 하지만, 이러한 엔티티 헤더는 종종 컨텍스트와 같은 요청 헤더라 불립니다."*

<br><br>

### Response Header

HTTP 응답 시 사용되는 헤더

**예시**

```http
cache-control: max-age=0, no-cache, no-store, must-revalidate, private
content-length: 35
content-type: application/json; charset=utf-8
date: Wed, 30 Mar 2022 13:42:04 GMT
expires: Wed, 30 Mar 2022 13:42:04 GMT
referrer-policy: same-origin
server: gunicorn
vary: Cookie
via: 1.1 5062540db61fe5bfa0e8709e57809128.cloudfront.net (CloudFront)
x-amz-cf-id: _OoJKUVILmX4SgHZ06Z3FJV5J8HcjgsgJtdsh0Yw26XjoLfxe74-Ng==
x-amz-cf-pop: ICN51-C2
x-cache: Miss from cloudfront
x-content-type-options: nosniff
x-frame-options: DENY
```

**엄격하게 본다면, (브라우저 확인 시) Response Header 에 포함된 헤더 모두가 Response Header 는 아님에 주의**

예를 들면 아래와 같은 것들은 `Entity Header` 이다.

- Content-Length
- Content-Encoding
- Content-Type

> *"응답에 나타나는 모든 헤더가 응답 헤더인것은 아닙니다. 예를 들어, Content-Length 헤더는 응답 메시지 바디의 크기를 참조하는 entity header입니다. 그러나 이러한 엔티티 요청은 보통 컨텍스트에서 응답 헤더로 불립니다."*

<br><br>

## HTTP 종류

> https://developer.mozilla.org/ko/docs/Web/HTTP/Headers
