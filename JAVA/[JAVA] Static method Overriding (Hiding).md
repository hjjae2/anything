### Static method Overriding

JVM은 메서드를 호출할 때, 객체 메서드(instance method)의 경우 (런타임 시에) 해당 메서드를 구현하고 있는 실제 객체를 찾아 호출한다.

하지만 스태틱 메서드(static method)에 대해서는 실제 객체를 찾는 작업을 하지 않는다. 따라서 Static method의 경우 컴파일 시점에 선언된 타입의 메서드를 호출한다. 

즉 Static method 에 대해서는 오버라이딩이 적용되지 않는다. 또, 이때에는 Overriding이 아닌 Hiding 이라고 부른다고 한다.

<br>

### 예시 코드

```java
public class A{
    public static void test() {
        System.out.println("A test()");
    }
}

class B extends A{
    public static void test() {
        System.out.println("B test()");
    }
}

public class Test {
    public static void main(String[] args) {
        A a = new B();
        System.out.println(a.test()); 
        // 출력: A test()

        B b = new B();
        System.out.println(b.test());
    }
}
```

<br>

### 참고
- https://blog.naver.com/gngh0101/221206214829
- https://hashcode.co.kr/questions/358/왜-자바에서-static메소드의-오버라이딩을-허용하지-않는걸까요