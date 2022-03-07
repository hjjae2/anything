## Throwable

> *" The Throwable class is the superclass of all errors and exceptions in the Java language. Only objects that are instances of this class (or one of its subclasses) are thrown by the Java Virtual Machine or can be thrown by the Java throw statement. Similarly, only this class or one of its subclasses can be the argument type in a catch clause. For the purposes of compile-time checking of exceptions, Throwable and any subclass of Throwable that is not also a subclass of either RuntimeException or Error are regarded as checked exceptions. "*

1. 오직 Throwable, Throwable 의 하위 클래스만 JVM, Java 에 의해 던져질 수 있음 (can be thrown)

2. 오직 Throwable, Throwable 의 하위 클래스만 catch 문법에 사용될 수 있음

3. Error, RuntimeException 을 제외한 Throwable 의 하위클래스는 CheckedException
- CheckedException : 컴파일 시점에 Exception 체킹 목적으로 사용됨

- class 임에 주의 (참고 : [why is java.lang.Throwable a class?](https://stackoverflow.com/questions/2890311/why-is-java-lang-throwable-a-class?answertab=votes#tab-top))


```java
public class Throwable implements Serializable {
    ...
}
```


<br>

## Error

시스템(서버) 레벨, HW 레벨의 오류

수습할 수 없는 심각한 문제

- VirtualMachineError
- OutOfMemoryError
- InternalError
- StackOverflowError
- UnknownError

<br>

## Exception 

### CheckedException

예외 처리 필수

컴파일 시 체크

- IOException
- InterruptedException
- ClassNotFoundException
- NoSuchMethodException
- NoSuchFieldException


### RuntimeException (UncheckedException)

- NullPointerException
- ArithmeticException
- IllegalArgumentException
- NumberFormatException