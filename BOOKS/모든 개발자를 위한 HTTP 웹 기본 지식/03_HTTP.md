## 요약

### HTTP (HyperText Transfer Protocol)

1. Client - Server 구조
   1. 클라이언트 - 서버의 분리는 역할/책임 분리의 의미에서도 중요한 것
2. 비상태성 (Stateless)
   1. 서버 확장(Scale Out) OK
   2. (클라이언트의 요청에 대해서) 항상 다른 서버가 처리 OK
   3. 클라이언트쪽에서 조금 불편 (서버 측에서 정보를 기억하는게 아니니까)
   4. 상태가 필요한 경우 : 대표 예시 -> 로그인
      1. **특정 서버에 구애받지 않게 독립적인(외부) 상태 저장소 사용**
3. 비연결성
   1. 연결을 유지할 경우, 수많은 클라이언트와 연결해야 하는 부담
   2. 서버 자원을 효율적으로 사용하기 위함
4. 단순


**\* HyperText 란?**

> " *하이퍼텍스트(Hypertext, 문화어: 초본문, 하이퍼본문)는 참조(하이퍼링크)를 통해 독자가 한 문서에서 다른 문서로 즉시 접근할 수 있는 텍스트이다.* " (출처 : https://ko.wikipedia.org/wiki/하이퍼텍스트)

- HTML
- TEXT
- ...

<br>


### HTTP 역사

**HTTP/0.9**

- 1991년
- GET 만 지원
- HTTP 헤더 X


**HTTP/1.0**

- 메서드(GET, HEAD, POST), 헤더 추가 -> 실제로 사용되기 시작
- 요청마다 TCP 연결/종료
  - Persistent Connection(Keep-Alive) 위해서 'Connection 헤더'에 사용

**HTTP/1.1**
> TCP

- **Persistent Connection(Keep-Alive)** : Default
- **HTTP Pipelining : 이전 요청에 대한 응답을 기다리지 않고, 새로운 요청을 보낼 수 있음**
  - HTTP 1.0 
    - 요청1 - 응답1
    - 요청2 - 응답2
    - 요청3 - 응답3
  - HTTP 1.1
    - 요청1, 요청2, 요청3
    - 응답1, 응답2, 응답3
- **Header : Host 정보 추가**
  - VirtualHost 사용 가능하게 됨
- 캐시 헤더 ?? 캐시 제어 메커니즘 
- 콘텐츠 협상 ??
- PUT, DELETE, TRACE, OPTION

**HTTP/2**

> TCP

- 2015년
- 성능 개선

**HTTP/3**

- 진행중
- UDP 사용 (성능 개선)


<br>
<br>


### HTTP 메시지 구조

```
start-line (시작 라인)
header (헤더)
empty line (공백 라인, 필수)
message body (바디)
```

**HTTP 요청 메시지 예시**
```
GET /search HTTP/1.1  : method path(absolute-path[?query]) http-version 
HOST: www.naver.com

```

**HTTP 응답 메시지 예시**
```
HTTP/1.1 200 OK
Content-Type: ~  => field-name: field-value
Content-Length: ~ 

<html>
 ~~~
</html>
```

**HTTP 헤더**

: HTTP 전송에 필요한 모든 부가정보

**HTTP 바디**

: 실제 전송할 데이터 (HTML, 이미지, 영상, JSON 등의 byte로 표현할 수 있는 모든 것 = 거의 모든것이지 않을까)