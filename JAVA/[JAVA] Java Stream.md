---
layout: post
title: "Java :: Stream API"
author: "leehyunjae"
tags: ["java"]
---

> Stream 에 대해 정리해보기

### Stream

Java8 에서 java.util.stream 패키지에 Stream API 추가

다양한 데이터 소스(source)를 표준화된 방법으로 다루기 위한 기술

<br><br>

### Stream API 의 특징은 다음과 같다.

1. **데이터(컬렉션, 배열 등)을 표준화된/하나의 방법을 통해 연산, 조작할 수 있다.**
   - 데이터를 스트림으로 만들고 나면 표준화된(하나의) 방법으로 조작할 수 있다.
2. **내부 반복(internal interation) 을 통해 작업을 수행한다.**
3. **스트림은 일회용이다.**
   - 최종 연산(소모)을 통해 스트림이 끝났다면, 다시 스트림 생성/조작해야 한다.
4. **원본 데이터(original data)를 변경하지 않는다.**
5. **지연(Lazy) 연산을 통해 불필요한 연산을 피한다.**
6. **쉽게 병렬 처리 기능을 지원한다.**
   - `parallelStream()`, `parallel()` 등
7. **스트림의 처리 단계는 다음과 같다.**
   1. 스트림의 생성 
   2. 스트림의 중개/중간 연산(스트림의 변환 : filter, map, ...)
   3. 스트림의 최종 연산(스트림 사용, 스트림 요소 소모 : reduce, collect, ...)
   
   - 중간 연산 : 연산 결과 -> stream
   - 최종 연산 : 연산 결과 -> stream X, <u>스트림 소모하는 방식이라 단 한번만 가능</u>
8. **기본형(Primitive) 스트림을 지원한다.** (`IntStream`, `LongStream`, `DoubleStream` 등)
   - `Stream<Integer>` 대신 `IntStream` 을 사용함으로써 오토박싱&언박싱 비효율 제거
   - `Stream<T>` 보다 숫자와 관련된 유용한 기능을 더 제공 (`Stream<T>`는 숫자를 위한 것이 아니고 참조타입을 위한거니까)
   - 성능 개선에 고려할 수 있다.

**\* 스트림으로 연산(중개연산, 최종연산)을 할 때 대부분 '함수형 인터페이스 매개변수'를 갖는다. 즉, 람다식을 사용할 수 있다.**

<br>

```java
// 스트림을 생성하는 다양한 방법들
Collection.stream();

Stream.of();

Stream.iterate();

Stream.generate();

// 중개/중간 연산 종류들
filter();
distinct();
sort();
limit();
...

// 최종 연산 종류들
count();
forEach();
...
```

<br><br>

### Lazy 연산 vs Eager 연산

```java
List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
list.stream()
   .filter(i -> i<6) // --- (1)
   .filter(i -> i%2==0) // --- (2)
   .map(i -> i*10) // --- (3)
   .collect(Collectors.toList())

// 출처: https://dororongju.tistory.com/137 [웹 개발 메모장]
```

**Lazy 연산**

1. 1 번째 연산에 대해 계산 : (2)까지 계산 후 연산 종료
2. 2 번째 연산에 대해 계산 : (3)까지 계산
3. 3 번쨰 연산에 대해 계산 : (2)까지 계산 후 연산 종료
4. n 번째 연산에 대해 계산 : ...
5. 10 번째 연산에 대해 계산 : (1)까지 계산 후 연산 종료

<br>

**Eager 연산**

1. 1~10번째에 대해서 (1) 연산 실행
2. 1번에서 구한 요소들에 대해 (2) 연산 실행
3. 2번에서 구한 요소들에 대해 (3) 연산 실행

<br><br>

### 병렬 스트림

1. Stream API 에는 '병렬 스트림'을 생성할 수 있는 API 를 제공한다. 

2. 병렬 스트림이란, 각각의 스레드에서 처리할 수 있도록 Stream 의 요소(element)를 여러 chunk 로 분할한 스트림이다. 이 chunk 를 멀티코어 CPU 가 처리하도록 할당할 수 있다. (\* 병렬 스트림은 내부적으로 `ForkJoinPool` 을 사용한다.)

- \* 병렬 처리가 항상 향상된 성능으로 동작하는 것은 아니다.

- \* <u>어떤 알고리즘을 병렬화 하는 것보다 어떤 자료구조를 선택할 지 고민하는 것이 중요하기도 하다. (예를 들어, 불필요한 박싱/언박싱 제거)</u>

> 많은 글들에서 스트림은 "내부 반복" 을 통해 병렬 처리를 쉽게할 수 있다고 한다. 그렇다면 왜 내부 반복을 사용하면 병렬 처리를 쉽게 할 수 있을까? 내가 생각한 결론은 "병렬 처리를 위해 요소들을 chunk 단위로 분할할 때, 개발자가 직접 분할하지 않고 내부에서 알아서 분할해줄 수 있기 때문에" 이다.

<br>

**병렬 처리를 할 때 아래의 것들에 대해 고려해봐야 한다.**

1. 병렬 처리의 동작 방식에 대해 정확히 알고 사용하자.

2. 멀티코어 간의 데이터 이동 비용과 작업(로직)의 비용을 비교하자.

3. 공유 자원이 연관된 작업에 대해서는 병렬화의 이점이 없을 수 있다. (공유 자원에 대해 관계가 없을 때 사용하는 것이 바람직하다.)

4. 전체 스트림 파이프라인 처리 비용 = N * Q (N = 처리해야할 요소의 수, Q = 하나의 요소를 처리할 때 발생하는 비용)일 때, Q 가 높다면 병렬 스트림으로 성능을 개선할 수 있는 가능성이 있다. 
   - N 이 커도 개선할 수 있는 가능성이 있다.

5. '최종 연산' 의 병합 비용을 고려한다.
   - 병합 비용이 비싸다면, 이득이 없을 수 있다.

6. 소량의 데이터에서는 병렬 스트림의 이득이 크지 않다.

<br><br>

### Fork/Join Framework (포크/조인 프레임워크) 란?

Fork/Join 프레임워크는 작업을 작은 작업으로 분할하고, (sub task)각각의 결과를 합쳐 전체 결과를 만들도록 설계되었다.

서브태스크(sub task)를 ForkJoinPool의 작업 스레드에 분산하여 할당하는 ExcutorService 인터페이스를 구현한다.

<br><br>

### 참고

- [자바의 정석 - 기초편, 스트림, 스트림의 특징](https://www.youtube.com/watch?v=7Kyf4mMjbTQ&list=PLW2UjW795-f6xWA2_MUhEVgPauhGl3xIp&index=163)
- [스트림 API](http://tcpschool.com/java/java_stream_concept)
- [병렬 데이터 처리와 성능(병렬 스트림)](https://yongho1037.tistory.com/705)
- [Lazy Evaluation 이란?](https://dororongju.tistory.com/137)