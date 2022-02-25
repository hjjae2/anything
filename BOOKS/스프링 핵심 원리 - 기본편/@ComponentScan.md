### @Component 어노테이션이 달린 클래스들을 모두 찾아, 빈으로 등록

- `@Component`
- `@Controlloer` (@Component 갖고 있음) : Spring MVC 컨트롤러로 인식
- `@Service` (@Component 갖고 있음) : 특별한 기능 X
  - 비즈니스 로직 인식 용도
- `@Repository` (@Component 갖고 있음) : 데이터(DB) 계층의 에러를 스프링의 (통일된)예외로 변환
  - 여러 DB마다 에러가 다를 수 있다 -> 통일화
- `@Configuration` (@Component 갖고 있음) : 스프링 설정 정보로 인식

> **\* 어노테이션의 어노테이션을 인식 => Java의 기능 X, Spring 의 기능 O**

<br>

**basePackage : default(클래스 파일의 위치) 설정 권장**

<br>

**(아래와 같이)빈으로 등록할 때 의존관계가 있는 것들은 어떻게 해결?**

**--> 생성자 주입, 필드 주입, 세터 주입**


```java
// 수정 전

@Configuration
public class AppConfig {
    @Bean
    public MyService myService() {
        return new MyService(myRepository());
    }

    @Bean
    public MyRepository myRepository() {
        return new MyRepository();
    }
}
```

```java
// 수정 후

@Component
public class MyService {

    private final MyRepository myRepository;

    @Autowired
    MyService(MyRepository myRepository) {
        this.myRepository = myRepository;
    }
}
```

<br>

> includeFilters, excludeFilters 를 엄청 사용하지는 않는 듯 <br>
> 
> 단, excludeFilters 는 가끔 사용

<br><br>

### FilterType 옵션 

- ANNOTATION : Default, 어노테이션 인식
  - ex) `org.example.SomeAnnotation`
- ASSIGNABLE_TYPE : 지정한 타입 , 자식 타입 인식
  - ex) `org.example.SomeClass`
- ASPECTJ : AspectJ 패턴 사용
  - ex) `org.example..*Service+`
- REGEX : 정규표현식 사용
  - ex) `org\.example\.Default.*`
- CUSTOM : `TypeFilter` 인터페이스 구현/처리
  - ex) `org.example.MyTypeFilter`


<br><br>

### Bean 중복/충돌

> 자동 등록 : @Component by @ComponentScan
> 
> 수동 등록 : @Configuration (@Bean)

**자동 등록 vs 자동 등록**

-> 충돌 오류

<br>

**수동 등록 vs 자동 등록**

(SpringFramework) (오류 X) 수동 등록 우선순위 

**\* 단,(SpringBoot) 오류 O**

> (등록하는 Bean이 너무 많기 때문에) 실무에서는 수동 등록 / 자동 등록의 경우가 꼬이는 경우가 대부분이다. <br>
> -> 앱 실행 시 오류 발생하도록 정책 바꿈
> 
> `allow-bean-definition-overriding` 옵션 설정 가능