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

> orElse() vs orElseGet(), https://velog.io/@joungeun/orElse-vs-orElseGet