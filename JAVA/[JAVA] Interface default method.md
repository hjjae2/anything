
Java 8 부터 Interface 에서 default method 를 구현할 수 있게 되었다.


### 추상 클래스와의 차이점 ??

용도/목적이 다르다.
   - `abstract class` 는 말 그대로 '클래스' 이다. `extends` 하는 것이다. 멤버변수, 접근제어 등의 기능을 갖는다.
   - `interface` 는 말그대로 '인터페이스'이다. `implements` 하는 것이다.


<br><br>
### 인터페이스 다중 구현에서 Default 메서드가 중복일 때
A 라는 클래스가 Interface1, Interface2 를 다중 구현하고 있는데, Interface1, Interface2 가 동일한 이름의 default 메소드를 갖고있다면 어떻게 될까.

**결론 : (컴파일)에러가 난다.**

`A inherits unrelated defaults for watchTv() from types Man and Woman`

```java
// Man Interface
public interface Man {
    void playGame();

    default void watchTv() {
        System.out.println("Man is watching tv");
    }
}


// Woman Interface
public interface Woman {
    void playGame();

    default void watchTv() {
        System.out.println("Woman is watching tv");
    }
}

// A class
public class A implements Man, Woman{
    @Override
    public void playGame() {
        System.out.println("A is playing game");
    }
}
```

<br>

**해결 방법**

1. A class 중복이 되는 default 메서드(`watchTv()`)를 오버라이딩, 구현한다.

2. 인터페이스들 중 default 메소드를 제거하여 중복을 제거한다.