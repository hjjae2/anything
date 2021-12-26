### SSL Handshake 동작 원리 / 과정

**SSL Handshake의 목표**

1. <u>신뢰할 수 있는 서버(연결)</u>인지 확인합니다.
2. 통신 시 사용할 <u>암호화 알고리즘</u>을 결정합니다.

<br>

> 자세한 설명/그림은 [여기](https://nuritech.tistory.com/25)를 참고합니다.

<br>

**1. [Client 측] Client Hello**

Client -> Server 로 연결을 시도합니다. 이때, 아래의 내용을 포함합니다.

- 사용할 수 있는 암호화 알고리즘 (즉, 어떤 암호화 알고리즘을 사용할 수 있는지)
- SSL Protocol version 등등

<br>

**2. [Server 측] Server Hello / Certificate**

Server -> Client 로 응답합니다. 이때, 아래의 내용을 포함합니다.

- Client 가 보내준 암호화 알고리즘들 중 어떤 것을 사용할 것인지
- SSL Protocol version
- SSL 인증서 내용

<br>

**3. [Client 측] Server의 SSL 인증서 검증**

CA의 비밀키로 암호화된 서버의 SSL 인증서를, CA의 공개키로 복호화 시도합니다.

복호화가 정상적으로 되었다면 검증이 완료된 것으로 봅니다.

> \* CA의 공개키는 브라우저에 내장되어 있거나 인터넷상으로 다운로드 받아올 수 있다고 합니다.

<br>

**4. [Client 측] 대칭키 전달**

Client -> Server 로 대칭키를 전달합니다.

<u>대칭키를 전달할 때, Server의 공개키로 암호화하여 전달합니다.</u>

> \* Server의 공개키는 3번(Server의 SSL 인증서 검증/복호화)시점에서 얻은 것 입니다.

<br>

**5. [Server 측] SSL Handshake 종료**

서버의 비밀키를 통해 클라이언트가 전달한 대칭키를 획득합니다.

이로써 SSL Handshake는 종료됩니다. (통신할 준비가 완료되었음을 의미합니다.)

<br><br>

### 참고

[HTTPS 통신 원리 쉽게 이해하기 (Feat. SSL Handshake, SSL 인증서)](https://nuritech.tistory.com/25)
