### Interface default method


Java 8 부터 Interface 에서 default method 를 구현할 수 있게 되었다.

여기서 한가지 의문점이 생겼다.

그렇다면, 추상클래스와 점점 비슷해지는 거 아닌가..? 추상클래스와 똑같아지는 건 아닌가..?

또, 다음과 같은 경우도 궁금해졌다.

A 라는 클래스가 Interface1, Interface2 를 다중 상속하고 있는데, Interface1, Interface2 가 동일한 이름의 default 메소드를 갖고있다면 어떻게 될까..?

결론 : 에러가 난다.

```
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

이 경우 아래와 같은 컴파일 오류가 발생한다.

`A inherits unrelated defaults for watchTv() from types Man and Woman`

<br>

### 해결 방법

1. A class 에서 watchTv() 메소드를 구현한다. 즉, 인터페이스들에서 중복으로 구현되어 있는 default 메소드가 있다면, 이것을 구현해주어야 한다.

2. 인터페이스들 중 default 메소드를 제거하여 중복을 제거한다.