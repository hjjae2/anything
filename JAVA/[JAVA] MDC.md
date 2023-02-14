다음 내용을 참고하자.
- https://mangkyu.tistory.com/266
- https://bcho.tistory.com/1316
- https://logback.qos.ch/manual/mdc.html

![](../images/[JAVA]%20MDC_50.png)

> 출처: https://bcho.tistory.com/1316

<br>

요약하면, (ThreadLocal 기반으로 동작하며) CorrelationID + 메타 데이터를 Map 형태로 관리할 수 있다. 

관리되는 메타 데이터들을 로깅 시 함께 남겨 애플리케이션 로그 추적을 쉽게 한다. (`%X{placeholder}`  패턴으로 남길 수 있다.)