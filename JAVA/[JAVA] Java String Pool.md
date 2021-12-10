---
layout: post
title: "Java :: String Pool"
author: "leehyunjae"
tags: ["java"]
---

> String Pool 에 대해 이해해보기

<br>

### Java에서 String 객체를 생성하는 2가지 방법

1. new 연산자
   - `String str = new String("안녕하세요");`
2. String Literal
    - `String str = "안녕하세요"` 

두 방식 모두 Heap 에 생성되지만, 차이점도 존재한다고 한다.

<br>

### new 연산자

- (값이 같아도, 달라도) 서로 다른 주소(reference value)를 가진다.
- 즉, 계속해서 새롭게 생성한다.

```java
String str1 = new String("안녕하세요"); // Heap 영역에 "안녕하세요" 라는 객체가 새롭게 저장된다.
String str2 = new String("안녕하세요"); // Heap 영역에 "안녕하세요" 라는 객체가 새롭게 저장된다.
```

<br>

### String Literal

- (값이 같다면) 서로 같은 주소(reference value)를 가진 객체이다.
- 즉, 재사용된다.

> Heap 내부에 String Pool 이라는 공간이 있고, 생성한 문자열의 값(e.g. "안녕하세요")이 저장된다. 이후에 동일한 값(e.g. "안녕하세요")으로 생성되는 String은 이미 Stirng Pool에 있는 객체를 가르킨다.

```java
String str1 = "안녕하세요" // String Pool 에 "안녕하세요" 라는 애가 String pool 에 저장된다.
String str2 = "안녕하세요" // 위에서 생성된 "안녕하세요"를 가르킨다.
```
<br>

### 알아두기

Java6 이전 : `String Pool`은 `PermGen` 이라는 (fixed size)Space 안에 있었다고 한다. 사이즈가 고정되었고, 런타임에 동적으로 확장할 수도 없어서 `OutOfMemory` 가 발생할 여지가 컸다고 한다. (Garbage Collection 구조에도 적합하지 않았다고 한다.)

Java7 부터 : `String Pool`은 `Heap` 공간 안에 위치했고 (Heap 영역에 위치했다는 것은 Garbage collected 됨을 의미하기에) `OutOfMemory` 리스크가 줄었다고 한다.

또, String Pool의 사이즈는 1009 buckets -> 60013(size?) -> 65536(size?) 로 변경되었다고 한다.

> Prior to Java 7u40, the default pool size was 1009 buckets but this value was subject to a few changes in more recent Java versions. To be precise, the default pool size from Java 7u40 until Java 11 was 60013 and now it increased to 65536.

> (\+ 추가적으로 읽어보면 좋은 내용) Until Java 8, Strings were internally represented as an array of characters – char[], encoded in UTF-16, so that every character uses two bytes of memory.
> With Java 9 a new representation is provided, called Compact Strings. This new format will choose the appropriate encoding between char[] and byte[] depending on the stored content.
> Since the new String representation will use the UTF-16 encoding only when necessary, the amount of heap memory will be significantly lower, which in turn causes less Garbage Collector overhead on the JVM.

<br>

### 결론

**String Literal 을 사용하면 객체를 재사용할 수 있다.**

<br>

### 참고

- [Guide to Java String Pool](https://www.baeldung.com/java-string-pool)