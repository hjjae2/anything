## Reactive Programming

> 참고 
> - https://gngsn.tistory.com/223
> - https://en.wikipedia.org/wiki/Reactive_programming

**Keywords**

1. Data Stream
2. Functional Programming
3. Asynchronous
4. Declarative Programming Paradigm

Reactive programming is a **declarative programming paradigm** concerned with **data streams** and the propagation of change. 

> *" **Reactive Programming은 데이터 스트림을 비동기 처리하는 선언형 프로그래밍입니다.** 선언형 프로그래밍이란 기존의 명령형 프로그래밍 방식과 대비되는 새로운 프로그래밍 패러다임으로, 라인 단위의 프로그래밍 과정과 달리 특정 목적과 같이 무엇을 하는 지를 명시하여 개발하는 과정입니다. 리액티브 프로그래밍은 아래 3가지(Data Stream, Functional Programming, Asynchronous) 측면으로 기존 프로그래밍 방식의 문제점들을 해결합니다. "*
> 
> 출처: https://gngsn.tistory.com/223


<br>

## Change propagation algorithms

### Pull 

The consumer queries the observed source for values and reacts whenever a relevant value is available. 

The practice of regularly checking for events or value changes is commonly referred to as **polling**.

<br>

### Push

The consumer receives a value from the source whenever the value becomes available.

These values are self-contained, **e.g. they contain all the necessary information, and no further information needs to be queried by the consumer.**

> :star: 이 부분('필요한 정보는 모두 Push 한다'는 특징)이 Push-Pull 알고리즘과의 차이점이다. 


<br>

### Push-Pull

The consumer receives a change notification. **(Push part)**

However, the notification dose not contain all the necessary information. 

So the consumer needs to query the source for more information (after it receives the notification). **(Pull part)**

**This 'Push-Pull' method is commonly used when there is a large volume of data tha the consumers might be potentially interedted in.** (So in order to reduce throughput and latency, only light-weight notifications are sent.)

This approach also has the drawback that the source might be overwhelmed by many requests(query) for additional information after a notification is sent.