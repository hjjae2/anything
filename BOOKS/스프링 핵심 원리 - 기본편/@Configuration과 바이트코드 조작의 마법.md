### @Configuration, @Bean 을 통해 Bean 을 등록할 수 있다.

<br>

### CGLIB 이 나의 클래스를 상속한 Proxy 클래스(객체)를 생성하고, 이 클래스(객체)를 Bean 으로 등록

```java
if(클래스(객체) 이미 존제?) {
    return 객체 반환
}
else {
    obj = new 클래스();
    obj -> bean 으로 저장;
    return obj
}
```

<br>

### @Configuration 없이 @Bean 만 사용해도 Bean 으로 등록된다.

단, 이때는 CGLIB 이 Proxy 객체로 만들어주지 않느다. <br>
= Proxy 객체를 생성 X <br>
= 싱글톤을 보장 X <br>