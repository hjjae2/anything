## MQTT (Message Queueing Telemetry Transport)

> Telemetry : 원격 측정(법)

다음과 같은 상황에서 사용될 수 있는 메시지 송/수신 프로토콜이다.

(1) 작은 코드 공간에서 동작하기 위해서
(2) 제한된 네트워크 대역폭에서 동작하기 위해서 (ex: IoT)
(3) 대규모 트래픽 전송을 위해서

TCP/IP 위에서 동작하지만 굉장히 가볍고, 많은 통신 제약들을 해결해준다고 한다.

- MQTT 는 Bluetooth, Zigbee 처럼 별도의 모듈 / 별도의 대역폭을 갖는 통신 규약이 아닌, WiFi와 같은 인터넷을 통해 TCP/IP 기반의 메시지 송수신 프로토콜이다.
- (Message가) 가벼운 만큼, QoS(서비스 품질)에는 제약이 있다고 한다.
- IoT 환경에서 메인으로 사용되는 프로토콜이라고 한다.

> " 이러한 장점들 때문에 Facebook Messenger가 MQTT를 채택했고, 우아한형제들(배달의 민족 서비스 기업)에서도 중계 시스템 개선을 위해 MQTT를 도입하려 시도한 적이 있었다. (해당 이야기는 후술) "
> [[통신 이론] MQTT, MQTT Protocol (MQTT 프로토콜) 이란? - 1 (이론편)](https://underflow101.tistory.com/22)


<br><br>

### 구조

> Kafka 와 비슷하게, Pub/Sub (w. Broker) 구조인 것 같다.

```
pub --- broker(topic) --- sub
```

<br><br>

### MQTT vs Kafka


Kafka 의 경우,

1. 대용량의 데이터를 안전하게 disk에 저장하고, real-time or later 소비할 수 있도록 하는 것에 집중한다.
   - 참고 : https://stackoverflow.com/a/37407079
2. Pub/Sub Messaging + Streaming Processing, Data Integration, ...
   - MQTT : Pub/Sub (only)


기타 내용(차이점)은 [여기 (Apach Kafka and MQTT : Overview)](https://velog.io/@jamangstangs/Apach-Kafka-and-MQTT-Overview)를 참고한다.

<br><br>

### 참고

1. [[통신 이론] MQTT, MQTT Protocol (MQTT 프로토콜) 이란? - 1 (이론편)](https://underflow101.tistory.com/22)
2. [Apach Kafka and MQTT : Overview](https://velog.io/@jamangstangs/Apach-Kafka-and-MQTT-Overview)