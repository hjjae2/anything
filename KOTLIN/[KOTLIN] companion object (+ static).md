> Java에서 'static'은 다양한 이유로 사용할 수 있다. <br>
> 1. '메모리 관리'를 위해 <br>
> 2. '공유'를 위해 (클래스 선언 정보와 함께 메모리 영역에 올라간다.)

<br>

## companion object

> 코틀린에서는 static 키워드가 없다.
> 
> 이를 대체하기 위해 companion object 를 많이 사용한다.<br>
> (다른 말로 표현하면) (companion) object 를 통해 조금 더 (OOP와 적절한 방식으로)구현할 수 있다.

<br>

### 예시 코드

```kotlin
class MyClass {
   companion object {
        fun doSomething() {

        }
    }
}
```

```java
public final class MyClass {

   // 결국에 object 로 만들어서 사용하는 것인데, 이는 OOP 의 방향성에 아주 적절하다.
   // 이 부분이 매우 중요하다!!
   public static final MyClass.Companion Companion = new MyClass.Companion(null); 

   public static final class Companion {
      public final void doSomething() {
      }

      private Companion() {
      }

      public Companion() {
         this();
      }
   }
}
```

<br><br>

## (kotlin에서) static 이 없는 이유?

1. 'Singleton' object 로 구현할 수 있음
   - (java) static 은 object 가 아님 (OOP의 방향성과 맞지 않음)
   - (kotlin) 위의 예시 코드와 같이 (싱글톤)객체로 구현 가능
   - (추가적으로) static 은 모던 프로그래밍 언어에 적절하지 않다고 한다.

2. 가독성/생산성 측면 (Productive-Oriented)
   - (java) static 은 장황/지저분한 코드를 생산한다.
   - (kotlin) object 구문의 사용은 이 문제를 해결한다.
     - 조금 더 생산성, 가독성 측면에서 우수하다고 말한다.


<br><br>

## 참고
- [Kotlin - companion object로 static 메소드, 객체 정의하기](https://codechacha.com/ko/kotlin-static-and-companion/)
- [[kotlin] Companion Object (1) - 자바의 static 과 같은 것인가?](https://www.bsidesoft.com/8187)
- [Why does Kotlin remove "static" keyword?](https://phucynwa.com/programming/why-does-Kotlin-remove-static-keyword/)