## orElse(T other) vs orElseGet(Supplier<? extends T> other)

### 간단 요약

- `orElse()` : Optional null 여부에 상관없이 메서드를 호출하여 (메서드의) 값을 반환한다.

- `orElseGet()` : Optional null 인 경우에만 메스드를 호출(supplier.get() 호출)하여 (메서드의) 값을 반환한다.

- 메서드의 인자를 보면 명확한 차이점이 있다. (Supplier의 Lazy Evaluation) <br><u>(T other) vs (Supplier<? extends T> other)</u>

```java
/**
 * Return the value if present, otherwise return {@code other}.
 *
 * @param other the value to be returned if there is no value present, may
 * be null
 * @return the value, if present, otherwise {@code other}
 */
public T orElse(T other) {
    return value != null ? value : other;
}

/**
 * Return the value if present, otherwise invoke {@code other} and return
 * the result of that invocation.
 *
 * @param other a {@code Supplier} whose result is returned if no value
 * is present
 * @return the value if present otherwise the result of {@code other.get()}
 * @throws NullPointerException if value is not present and {@code other} is
 * null
 */
public T orElseGet(Supplier<? extends T> other) {
    return value != null ? value : other.get();
}
```

> _" orElse는 객체를 그대로 return 하는데 orElseGet은 Supplier 메소드를 받아서 return 한다. ... <u>결론적으로 두 개의 차이는 메소드를 파라미터로 넘길 때 실행시점에서 차이가 발생한다.</u> orElse()는 Optional 객체가 null이 아닐 때에도 메소드가 실행되고, orElseGet은 실행되지 않는다. "_

<br>

### 예시 코드

```java
public class Main {
    public static void main(String[] args) {
        String check = "check"; // or null
        String str1 = Optional.ofNullable(check).orElse(func());
        String str2 = Optional.ofNullable(check).orElseGet(Main::func);

        System.out.println(str1);
        System.out.println(str2);
    }

    public static String func() {
        System.out.println("func() 실행합니다.");
        return "func()";
    }
}
```

<br>


```text
// check 값이 non-null 일 때,
func() 실행합니다.
check
check

// check 값이 null 일 때,
func() 실행합니다.
func() 실행합니다.
func()
func()
```

<br>

### 참고

- [orElse() vs orElseGet()](https://velog.io/@joungeun/orElse-vs-orElseGet)
- [자바8 Optional 3부: Optional을 Optional답게](https://www.daleseo.com/java8-optional-effective/)