---
layout: post
title: "Java :: Callable vs Runnable"
author: "leehyunjae"
tags: ["java"]
---

> Callable, Runnable 에 대해서 이해해보기

<br>

Java는 Multi-Threading 이라는 기술(기능)을 제공한다.<br>
`Runnable`, `Callable` 인터페이스를 통해 Multi-Threading 기능을 설계할 수 있다.

- `Runnable` : Multi-Threading 을 위해 제공되는 interface 이다.
- `Callable` : `Runnable`의 개선된 버전이다. (자바 5에서 추가)

> 두 인터페이스 모두 Multi-Threading 을 위한 것이다. 

|특징|Runnable|Callable|
|-|--------|---------|
|`RETURN`|리턴 타입 X|리턴 타입 O|
|`EXCEPTION`|Exception 발생 X|Exception 발생 O|
|`비고`|Multi-Threading 을 위해 제공되는 interface 이다.<br>`Thread`, `ExecutorService` 클래스 등을 통해 사용될 수 있다.|`Runnable`의 개선된 버전이다. (자바 5에서 추가)<br>`ExecutorService`, `FutureTask` 클래스 등을 통해 사용될 수 있다.|

<br>

### Callable

```java
/**
 * 1. Return 타입이 존재한다.
 * 2. Exception 을 발생시킨다.
 */
public interface Callable<V> {
    V call() throws Exception;
}
```

<br>

```java
public Example {
    public class MyCallable implements Callable<String> { // (String)Return Type 이 존재한다.
        @Override
        public String call() throws Exception { // Exception 을 발생시킨다.
            return "Hello!";
        }
    }
}
```

<br>

### Runnable

```java
/**
 * 1. Return 타입이 없다.
 * 2. Exception 이 없다.
 */
public interface Runnable {
    public abstract void run();
}
```

<br>

```java
public Example {
    public class MyRunnable implements Runnable {
        @Override
        public void run() { // (void)Return Type 이 없고, Exception 을 발생시키지 않는다.
            System.out.println("Hello!");
        }
    }
}
```

<br><br>

### 참고

- [Java - Runnable과 Callable의 차이점 이해하기](https://codechacha.com/ko/java-callable-vs-runnable/)
- [Runnable vs Callable in Java](https://www.baeldung.com/java-runnable-callable)