## Overview

두 가지 핵심 인터페이스(패키지)

**카프카에 메시지 발행**

`reactor.kafka.sender.*`
- `reactor.kafka.sender.KafkaSender`

**카프카의 메시지 컨슘**

`reactor.kafka.receiver.*`
- `reactor.kafka.receiver.KafkaReceiver`

<br><bR>

## KafkaSender

`KafkaSender` 는 thread-safe 하다. 여러 스레드와 공유하여 처리할 수 있다.

카프카로 메시지를 전송할 때 사용하는 `KafkaProducer` 하나와 연결된다.

`KafkaSender` 는 sender 설정 옵션(`reactor.kafka.sender.SenderOptions`) 인스턴스로 만든다.
- `KafkaSender` 를 만든 후엔, `SenderOptions` 를 수정해도 `KafkaSender` 에 반영되지 않는다.
  - 동시에 전송할 수 있는 최대 메시지 수(max inflight)와 같은 `KafkaSender` 전용 설정 옵션도 `KafkaSender` 인스턴스를 만들기 전에 미리 설정할 수 있다.
  - `KafkaSender` 를 생성하기 전에, `SenderOptions` 인스턴스에 Key, Value 의 Serializer 를 설정해야 한다.
- Broker(list), Serializer 와 같은 프로퍼티(속성)는 `KafkaProducer` 로 전달된다.
  - 이런 프로퍼티는 `SenderOptions` 인스턴스 생성 시점에 만들어도 되고, 인스턴스 생성 후에 setter 메서드 (`SenderOptions#producerProperty`)를 사용해도 된다.



```java
Map<String, Object> producerProps = new HashMap<>();
producerProps.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
producerProps.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, IntegerSerializer.class);
producerProps.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);

SenderOptions<Integer, String> senderOptions =
    SenderOptions.<Integer, String>create(producerProps)       // 1 : KafkaProducer 에서 사용할 옵션 값들을 설정한다.
                 .maxInFlight(1024);                           // 2 : KafkaSender 에서 사용할 옵션 값들을 설정한다.
```

> 필요한 옵션을 `SenderOptions` 에 정의했다면, 이 옵션으로 `KafkaSender` 인스턴스를 만들 수 있다.

```java
KafkaSender<Integer, String> sender = KafkaSender.create(senderOptions);
```

이제 `KafkaSender` 를 통해 카프카에 메시지를 보낼 수 있다. 
- **내부 `KafkaProducer` 인스턴스는 첫 번째 메시지를 보낼 준비가 되면 그때서야 생성된다. (lazy)**
- 위 코드 시점에서는 `KafkaSender` 인스턴스는 생성되었지만, 카프카에 연결되지는 않았다.

<br>

### SenderRecord

> 대략적으로 다음과 같은 느낌인 것 같다. <br>
> ProducerRecord + SendResult + correlation 메타데이터

레코드(ProducerRecord)와 전송 결과(SendResult)를 매칭하기 위한 별도의 correlation 메타데이터를 추가한 클래스이다.

`ProducerRecord` 는 카프카로 보낼 Key/Value 쌍과, (메시지를 전송할) 카프카 토픽명을 갖고 있다.
- 파티션을 지정해도 되고, 지정하지 않으면 설정에 있는 파티셔너에 의해 알아서 선택된다.
- 타임스탬프를 지정해도 되고, 지정하지 않으면 프로듀서가 현재 타임스탬프를 할당한다.

`SenderRecord` 의 'correlation 메타데이터'는 카프카로 전송되지 않지만, send 연산자가 완료/실패되었을 때 해당 레코드를 보내고 받은 결과가 `SendResult`에 담긴다.
- 다른 파티션에 보낸 전송 결과가 중간에 끼어들 수도 있기 때문에, 이 correlation 메타데이터로 (해당 레코드와 매칭해서) 올바른 레코드(전송 결과)를 찾을 수 있다.

<br>

### 예시


```java
Flux<SenderRecord<Integer, String, Integer>> outboundFlux =
    Flux.range(1, 10)
        .map(i -> SenderRecord.create(topic, partition, timestamp, i, "Message_" + i, i));

sender.send(outboundFlux)
      .doOnError(e-> log.error("Send failed", e))   // 1 : 카프카 전송(send)에 실패한 경우 에러를 처리한다.
      .doOnNext(r -> System.out.printf("Message #%d send response: %s\n", r.correlationMetadata(), r.recordMetadata())) // 2 : 카프카에서 결과를 받으면 처리한다.
      .subscribe(); // 3 : subscribe 를 통해 레코드(record)를 카프카로 전송하는 실제 플로우를 트리거한다.
```

카프카가 보낸 응답에는 record 를 전송한 파티션, 오프셋 정보들이 있다.

레코드들을 여러 파티션에 전송했다면, 다른 파티션의 응답이 중간에 끼어들 수 있다.


<br><br>

## Error Handling

```java
public SenderOptions<K, V> stopOnError(boolean stopOnError);
```

`SenderOptions#stopOnError()` 는 레코드 하나라도, 설정한 재시도 횟수만큼 카프카에 retry 한다. 

그럼에도 실패하면, 
- 전송 시퀀스를 즉시 실패시킬건지
- 모든 레코드를 처리할 때 까지 기다릴 건지를 지정한다.

`ProducerConfig#ACKS_CONFIG`, `ProducerConfig#RETRIES_CONFIG` 와 함께 설정해 원하는 서비스 품질을 구현할 수 있다.

```java
<T> Flux<SenderResult<T>> send(Publisher<SenderRecord<K, V, T>> outboundRecords);
```

`stopOnError` 가 false 이면, 각 레코드를 전송할 때마다 성공/에러 응답을 반환한다.

- 에러 응답을 받았을 때 : 카프카가 전송에 실패한 이유를 SendResult#exception() 에서 조회할 수 있다.
- **Flux는 `outboundRecords`에 발행한 모든 레코드를 전송해보고, 에러로 종료된다.**
  - `outboundRecords` 가 종료하지 않는 `Flux`라면 send 연산자는 사용자가 직접 `SenderResult` Flux를 취소할 때까지 계속해서 레코드를 전송한다.

<br>

`stopOnError` 가 true 이면, 첫 번째로 전송에 실패(설정한 retry 도 실패)했을 때 응답을 반환한다. (`SenderResult` Flux는 에러 발생 즉시 종료된다.)
- 여러 개가 전송 중이었을 때 : 일부 메시지는 카프카에 전달되었을 수도 있다.

<br><br>

...

<br><br>

## Kafka Receiver

`KafkaReceiver` 는 카프카 토픽에 저장된 메시지를 컨슘한다.

- `KafkaReceiver` 인스턴스는 단일 `KafkaConsumer` 인스턴스와 연결된다.
- 내부 `KafkaConsumer`도 thread-safe 하지 않기 때문에, `KafkaReceiver`도 thread-safe 하지 않다.

`KafkaReceiver` 는 `reactor.kafka.receiver.ReceiverOptions` 인스턴스로 만든다. 

- KafkaReciever 인스턴스를 만든 후엔, ReceiverOptions 를 수정해도 KafkaReceiver 에 반영되지 않는다.
- Broker(list), Deserializer 와 같은 `KafkaConsumer` 관련 속성들은 `KafkaConsumer`에 전달된다.
  - 이런 프로퍼티는 인스턴스 생성 전에 설정해도 되고, 생성 후에 setter 메서드(`ReceiverOptions#consumerProperty`)를 사용해도 된다. 
- `KafkaReceiver` 전용 설정 옵션(ex: 토픽 설정)은 `KafkaReceiver` 인스턴스를 만들기 전에 추가되어야 한다.

<br>

```java
Map<String, Object> consumerProps = new HashMap<>();
consumerProps.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
consumerProps.put(ConsumerConfig.GROUP_ID_CONFIG, "sample-group");
consumerProps.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, IntegerDeserializer.class);
consumerProps.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);

ReceiverOptions<Integer, String> receiverOptions =
    ReceiverOptions.<Integer, String>create(consumerProps)         // 1 : KafkaConsumer 에 제공할 프로퍼티를 지정한다.
                   .subscription(Collections.singleton(topic));    // 2 : 구독할 토픽을 설정한다. (KafkaReceiver 옵션)

```

```java
Flux<ReceiverRecord<Integer, String>> inboundFlux =
    KafkaReceiver.create(receiverOptions)
                 .receive();
```

**내부 `KafkaConsumer` 인스턴스는 inbound-flux 를 구독하면 그때가서 생성된다. (lazy)**

<br>

### ReceiverRecord

> 대략적으로 다음과 같은 느낌인 것 같다. <br>
> ConsumerRecord + ReceiverOffset (<- 커밋 기능을 제공하는 객체)

인바운드 메시지는 `ReceiverRecord` 로 표현한다.

**확인되지 않은 오프셋은 커밋하지 않으므로, 메시지를 처리하고 나면 반드시 오프셋을 처리했음을 알려야 한다. (acknowledge)**

커밋 인터벌(commit-interval), 커밋 배치 사이즈(commit-batch-size)를 설정했다면, 확인된(acknowledged) 오프셋을 주기적으로 커밋한다.

커밋 연산을 더 세밀하게 제어해야 할 땐, `ReceiverOffset#commit()` 을 사용해 수동으로 오프셋을 커밋할 수도 있다.

```java
inboundFlux.subscribe(r -> {
    System.out.printf("Received message: %s\n", r);           // 1 : 인바운드 메시지를 처리한다.
    r.receiverOffset().acknowledge();                         // 2 : offset을 커밋할 수 있게 레코드를 처리했음을 알린다. (acknowledge)
});
```

<br><br>

> 작성 중

<br><br>
### 참고

1. [Reactive Kafka Sender & Kafka Receiver](https://godekdls.github.io/Reactor%20Kafka/whatsnewinreactorkafka120release/)