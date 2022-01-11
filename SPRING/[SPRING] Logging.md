'Log' 는 소프트웨어의 행위(이벤트)를 기록하여, 문제가 발생했을 때 문제를 빠르게 파악할 수 있게 합니다. 또는 소프트웨어 자체를 모니터링할 수 있게 합니다.

Java 에서 log 관련 프레임워크는 아래와 같은 것들이 있습니다.

- Slf4j
- Log4j
- Logback
- Log4j2

<br><br>

### Slf4j (Simple Logging Facade For Java)

logger 인터페이스, 추상체 역할을 합니다. 즉, 다른 logging 프레임워크 구현체들의 인터페이스, 추상체 역할을 합니다.

예를 들어, Logback, Log4j2 의 Logger 클래스는 Slf4j Logger 인터페이스를 구현합니다.

```java
package ch.qos.logback.classic;

public final class Logger implements org.slf4j.Logger, ... {
    ...
}
```

Slf4j 인터페이스를 통해 쉽게 구현체(Logback, Log4j2, ...)를 변경할 수 있습니다.

<br><br>

### Log4j

가장 오래된 로깅 프레임워크입니다. 2015년에 개발이 중단되었습니다.

- Console / File 출력 등의 로깅이 가능합니다.
- xml, properties 를 통해 설정 가능합니다.

<br><br>

### Logback

log4j 보다 향상된 기능을 제공합니다. (필터링, 구성 파일 변경 후 자동 재적용 등)

<br><br>

### Log4j2

Log4j, Logback, Log4j2 중 가장 최근에 나온 프레임워크입니다.

Log4j, Logback 에 확장된 기능/성능을 제공합니다. (필터링, 구성 파일 변경 후 자동 재적용 등)

**특징**
- 비동기 로거(Async Logger) 지원 : (멀티 쓰레드일수록) 처리량 우수하고 대기시간이 짧다고 합니다.
- 람다식(Lazy Evaluation) 지원
- HTTP, Kafka 등에 출력 가능

SpringBoot 에서 사용 시에는 기본으로 의존되는 starter-logging 모듈을 exclude 하고, log4j2 의존성 주입을 해야 사용할 수 있습니다.

> 비동기 로거란?
> 
> 로그 발생(로깅 메서드 호출)과 로그 쓰기 작업을 분리합니다.
> 
> 1. Thread A는 로그를 발생시키고 Queue, Buffer 등에 로그 정보를 삽입합니다.
> 2. Thread B는 Queue, Buffer 에 쌓인 데이터를 꺼내어 쓰기 작업을 진행합니다. (Disk 쓰기라면 성능 차이가 더 심할 것 입니다.)
> 
> \* 주의할 점은 Queue, Buffer의 데이터가 유실될 가능성이 있습니다.
> 
> - 쓰기 작업 전에 Application 이 종료된 경우
> - Buffer, Queue 에 용량이 꽉 차 데이터가 제거, 삽입할 수 없는 경우

<br><br>

### Log Level

|LEVEL|설명|
|-|-|
|TARCE|DEBUG 보다 상세한 내용을 나타냅니다.|
|DEBUG|개발 시 디버깅 용도로 사용합니다.|
|INFO|정보 제공 용도로 사용합니다.|
|WARN|이후 에러가 될 수 있는 내용들을 알려줍니다.|
|ERROR|에러가 발생한 상태입니다.|
|FATAL|심각한 에러가 발생한 상태입니다.|

<br>

### Log Component

|요소|설명|
|-|-|
|Logger|- 로그의 주체, 로깅을 위한 클래스입니다.<br>- 출력할 메시지를 Appender에 전달합니다.<br>- LogLevel, Appender 를 설정합니다.<br>- 패키지, 애플리케이션 별로 설정할 수 있습니다.|
|Appender|- '어디'에 출력할 지 결정합니다. (Console, File 등)|
|Layout|- 어떤 형식(패턴)으로 출력할 지 결정합니다.|
