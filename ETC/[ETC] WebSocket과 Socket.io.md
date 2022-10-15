
> [WebSocket과 Socket.io](https://d2.naver.com/helloworld/1336) 내용을 정리합니다.

## 웹 소켓의 등장 배경

전형적인 브라우저 상의 동작 방식, 서비스 제공 방식은 **(1) 클라이언트 요청 -> (2) 서버 응답 (Client-Server 구조)** 의 방식에서 벗어나지 않았다. 

(시간이 지남에 따라, 더 나은 상호작용을 위해)Polling, Long Polling 등의 방법을 사용했지만 모두 클라이언트가 요청을 보내고 서버가 응답하는 **'단방향 통신'** 이다. (= 상호작용하는 서비스를 만들기 위해 복잡하고 어려운 코드를 구현해야 했다.)

보다 쉽게 상호작용할 수 있는 서비스, 기능을 제공하기 위해 **'양방향 통신'** 이 필요했고, WebSocket이 등장했다.

<br><br>

## 웹 소켓 프로토콜

> *표준 WebSocket API는 W3C에서 관장하고, 프로토콜은 IETF에서 관장한다.*

- 웹 소켓은 다른 HTTP 요청과 마찬가지로 80포트를 사용한다.
- Upgrade 헤더를 포함하여 웹 서버에 요청한다. (*Upgrade: WebSocket*)
- 클라이언트(브라우저), 웹 서버 모두 WebSocket 기능을 지원해야 한다.

아래와 같은 과정으로 WebSocket 핸드쉐이킹이 이뤄진다.

1. (브라우저) "Upgrade: WebSocket" 헤더 + 랜덤 키를 서버에 보낸다.
2. (서버) 이 키(랜덤 키)를 바탕으로 토큰을 생성한 후 브라우저에 보낸다.

핸드쉐이킹 후 **Protocol Overhead 방식**으로 웹 서버와 브라우저가 데이터를 주고 받는다.

이러한 방식을 사용하기 때문에 방화벽이 있는 환경에서도 무리 없이 WebSocket을 사용할 수 있다고 한다.

> Protocol Overhead 방식 <br>
> - 여러 TCP 커넥션을 생성하지 않고, 하나의 80 포트 TCP 커넥션을 이용한다.
> - 별도 헤더 등으로 (논리적인 데이터 흐름 단위를 이용하여) 여러 개의 커넥션을 맺는 효과를 낸다.

<br><br>

## 웹 소켓 지원 브라우저

![](../images/[ETC]%20WebSocket_15.png)

> https://caniuse.com/

<br><br>

## Socket.io

Socket.io는 JavaScript를 이용해서 브라우저 종류에 관계없이 실시간 웹을 구현할 수 있도록 한 기술이다. 

WebSocket, FlashSocket, Ajax Long Poliing, Ajax Multi part Streaming, IFrame, Json polling 등을 하나의 API로 추상화한 것이다. (<-- 이 문장을 보니 조금 이해가 되었다.) 

즉, 브라우저와 웹 서버의 종류와 버전을 파악하여 가장 적절한 기술을 선택하여 사용하는 방식이다. (예를 들어, 브라우저에서 FlashSocket을 사용할 수 있으면 사용하고, 사용할 수 없으면 Ajax Long Polling 을 사용)

Socket.io는 표준 기술이 아니고 Node.js 모듈로서 오픈소스다.

참고 : https://socket.io