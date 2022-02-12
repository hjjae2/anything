## HW 관점에서의 Thread

**CPU, Core, Thread**

- core : (CPU 내의) 물리적인 코어
- thread : 논리적인 코어
  - "동시에 실행가능한 스레드 개수"

<br>

**예시**

- CPU 2개 4코어 8스레드 -> 8개의 작업을 동시에 처리할 수 있음

<img src="./../images/2CPU%204Core%208Thread.png" width="45%">

<br><br>

## SW(Java) 관점에서의 Thread

> 이전부터 (HW 관점에서의)1코어 1스레드 환경에서, Java 는 n 개의 스레드를 사용할 수 있었음 -> **'동시성'과 관련**

- '동시성' : 여러 개의 작업들이 짧은 시간 내에 번갈아 처리됨 -> 동시에 처리되는 것처럼 보여짐
  - 컨텍스트스위칭 발생
- '병렬성' : 작업들이 병렬적으로 처리되는 것


<br>

**Java Thread 는 '동시성'을 가지는 것**

- N개의 Java Thread 가 (HW)Thread 를 번갈아 가며 사용/작업하는 것

- (어느 한 순간에) Java Thread 의 최대 병렬 작업 개수 = HW Thread 수
- 하지만 '동시성'의 성질을 갖고 있기에, 모든 Java Thread가 동시에 처리되는 것으로 보임

> 그래서 'Java Multi-Thread' 에서는 '병렬 프로그래밍' 이 아닌, '동시성 프로그래밍' 이라고 함

<br>
<br>

**(Java 에서) 동시성 Thread 성질을 통해 얻을 수 있는 것**

> == '동시성' 의 이점

- 비용이 크게 소요되는 작업 과 비용이 적게 소요되는 작업이 있을 때, 비용이 크게 소요되는 작업을 다 끝날때 까지 기다리지 않을 수 있음
  - 번갈아 가면서 실행하기 때문에, 위의 상황에서의 비효율성을 해소할 수 있음


<br><br>

## Java 에서 Multi-Threading 의 동기화를 지원하기 위한 방법

1. volatile
2. synchronized
3. Atomic (CAS)

<br>

### volatile

**CPU 캐시가 아닌, 메인 메모리(main memory)에 값 읽고/쓰기**

- '변수의 가시성' 문제를 해결

- CPU 캐시(각각의 HW Core)에 저장하는 것을 방지 -><br>
메인 메모리에 저장하도록 하여 여러 개의 Thread 가 동일한 값을 참조할 수 있도록 함.

**여러 개의 Thread 중 (동시에)하나의 쓰레드만 쓰기 작업을 보장한다면, volatile 로 동기화 문제 해소 가능**

> 변수의 값은 CPU 메모리와 메인 메모리에 저장된다.
> 
> 이 값을 CPU 메모리인지 메인 메모리에서 가져오는 지 알 수가 없다는 문제가 변수의 가시성 문제이다.
> 
> 출처: https://mygumi.tistory.com/112


<br>

### syncronized

**Lock 획득/해제 방식으로 동작.**

**주의할 점은, 메소드에 걸었을 경우 해당 인스턴스(객체)에 lock 을 거는 것**<br>
- `a()`, `b()` 메서드가 있을 때 Thread1가 `a()` 에 걸면 다른 스레드에서 `b()` 를 호출할 수 없음<br>
- `lock 재진입` 개념이 가능 (= `a()` 에서 lock 을 얻었으면, `b()` 함수도 바로 호출 가능)


```java
class MyObject {
    public synchronized int a() {
        ...
    }
    public synchronized int b() {
        ...
    }
}
```

```java
class MyObject {
    public int a() {
        synchronized(this) {
            ...
        }
    }
    public int b() {
        synchronized(this) {
            ...
        }
    }
}
```

**이유?**

데드락을 방지하기 위해서 인듯 함.
<br>
a() 함수 내에서 b() 함수 호출하고,<br>
b() 함수 내에서 a() 함수 호출하면 데드락이 발생

<br>

**단, static method 에 대해서는 클래스에 lock 을 건다.**

static method 는 아래와 같다고 함.

```java
class MyObject {
    public static synchronized int a() {
        ...
    }
    public static synchronized int b() {
        ...
    }
}

class MyObject {
    public static int a() {
        synchronized(MyObject.class) {
            ...
        }
    }
    public static int b() {
        synchronized(MyObject.class) {
            ...
        }
    }
}

```

<br>

**주의!!**

**static lock <-> 일반 lock 은 lock 객체가 서로 다름(class vs instance) -> 동기화 처리 되지 않음**

<br>

**참고**

```java
public class MyLock {

  private boolean isLocked = false;

  public synchronized void lock() throws InterruptedException{
    while(isLocked){
      wait();
    }
    isLocked = true;
  }

  public synchronized void unlock(){
    isLocked = false;
    notify();
  }
}
```

> 
<br>

### AtomicXXX (CAS)

**lock-free 하되 동기화를 보장하기 위해 사용 (CAS 방식을 통해 동기화를 보장)**

> (대게) 동기화 중 가장 빠름

(값 set 과 같은 어떤 행위를 할 때) 내가 갖고 있었던 '값'(혹은 메모리 값) 과 기대 값이 같은지 비교.<br>
-> 같다면, 안전하다고 판단하여 행위를 수행<br>
-> 다르다면, 정상적이지 않다고 판단하여 처리하지 않음.

> " 그래서 Non-blocking한 방법, Lock-Free한 방법으로 동기화 문제를 해결하기 위한 방법이 바로 Atomic연산이다.<br>
> 그리고 이 동작의 핵심 원리는 CAS(Compare And Swap)에 있다. "
> 
> 출처 : https://wannabe-gosu.tistory.com/29


```java
public boolean cas(int expectedValue, int newValue) {
    if(this.myValue != expectedValue) {
        return false;
    }
    this.myValue = newValue;
    return true;
}
```

```java
public class AtomicReference<V> implements Serializable {

    ...

    public final V updateAndGet(UnaryOperator<V> updateFunction) {
        V prev = this.get();
        V next = null;
        boolean haveNext = false;

        while(true) {
            if (!haveNext) {
                next = updateFunction.apply(prev);
            }

            if (this.weakCompareAndSetVolatile(prev, next)) {
                return next;
            }

            haveNext = prev == (prev = this.get());
        }
    }

    ...
}
```

<br><br>

### 참고

- https://javaplant.tistory.com/23
- https://badcandy.github.io/2019/01/14/concurrency-01/