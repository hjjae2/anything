### Static method overiding ?

JVM 이 메서드를 호출할 때, 객체 메서드(instance method)의 경우 (런타임 시에) 해당 메서드를 구현하고 있는 실제 객체를 찾아 호출한다. (다형성)

하지만 (컴파일러, JVM 의 경우) 스태틱 메서드(static method)에 대해서는 실제 객체를 찾는 작업을 하지 않는다. 따라서 Static method의 경우 컴파일 시점에 선언된 타입의 메서드를 호출한다. (즉 Static method 에 대해서는 다형성(오버라이딩)이 적용되지 않는다.)

<br>

### Hiding

아래와 같은 코드를 Hiding 이라고 한다. 이것은 새로운 메서드를 추가하는 개념이다.

```
public class A{
    public static void test() {
        System.out.println("A test()");
    }
}

class B extends A{
    public static void test() {
        System.out.println("A test()");
    }
}
```

> References
> 1. https://blog.naver.com/gngh0101/221206214829
> 2. https://hashcode.co.kr/questions/358/왜-자바에서-static메소드의-오버라이딩을-허용하지-않는걸까요