### Optional API : orElse vs orElseGet

`orElse(객체)` : Optional null 여부에 상관없이 객체를 return 한다.

`orElseGet()` : Optional null 인 경우에 객체를 return 한다.

> _" orElse는 객체를 그대로 return 하는데 orElseGet은 Supplier 메소드를 받아서 return 한다. ... 결론적으로 두 개의 차이는 메소드를 파라미터로 넘길 때 실행시점에서 차이가 발생한다. orElse()는 Optional 객체가 null이 아닐 때에도 메소드가 실행되고, orElseGet은 실행되지 않는다. "_

<br>

```
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

**예제 코드**

```
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

```
// if str is null, length is 0
// if str is not null, length will be str's length

String str = null; // str = "str";
int length = Optional.ofNullable(str).map(String::length).orElse(0);
System.out.println(length);
```

<br>

```
public static void main(String[] args) {
    List<String> strs = Arrays.asList("s1", "s2", "s3");

    int length = getAsOptional(strs, 3).map(String::length).orElse(0);
    System.out.println(length);
}

public static <T> Optional<T> getAsOptional(List<T> list, int index) {
    try {
        return Optional.of(list.get(index));
    } catch (ArrayIndexOutOfBoundsException e) {
        return Optional.empty();
    }
}
```

<br>

```
public static void main(String[] args) {
    List<String> strs = Arrays.asList("s1", "s2", "s3");
    int length = getAsOptional(strs, 2).map(String::length).orElse(0);
    getAsOptional(strs, 2).ifPresent(str -> System.out.println(str)); // 출력: "s2"
    getAsOptional(strs, 3).ifPresent(str -> System.out.println(str)); // 출력: x
}

public static <T> Optional<T> getAsOptional(List<T> list, int index) {
    try {
        return Optional.of(list.get(index));
    } catch (ArrayIndexOutOfBoundsException e) {
        return Optional.empty();
    }
}
```

<br>

**[참고]**

```
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

> orElse() vs orElseGet(), https://velog.io/@joungeun/orElse-vs-orElseGet
> 자바8 Optional 3부: Optional을 Optional답게, https://www.daleseo.com/java8-optional-effective/