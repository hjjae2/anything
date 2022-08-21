# Garbage Collection

사용 중이지 않은 객체(= Garbage)를 식별하여 정리(= 메모리 해제)

## 종류

- Serial GC
- Parallel GC
- ...
- CMS(Concurrent Mark Sweep) GC
- G1GC
- ZGC

<br>

## Mark & Sweep

### Mark

사용되는 메모리(객체)와 사용되지 않는 메모리(객체)를 식별하는 작업 (= reachability 를 판별하는 작업)

<br>

**root set(객체에 대한 최초의 참조) 가 될 수 있는 요소** 
- Java Stack (지역변수, 파라미터 등)
- Java Native Interface(= Native Stack)
  - JNI에 의해 생성된 객체들이 있을 때
- Method(Staic) Area (정적 변수)

root set으로부터 시작해서 참조로 연결되어 있는 모든 객체들을 탐색한다.

하나의 객체에 대한 참조의 개수, 타입은 다양하게 될 수 있다. (주로)참조의 타입에 따라 해당 객체의 reachability가 결정된다.

<br>

**reachability 의 종류 (5가지)**

|종류|설명|
|-|-|
|**strongly reachable**|root set부터 시작해서 해당 객체에 도달하기까지 어떠한 reference object도 없다.|
|**softly reachable**|(strongly reachable 객체가 아닌 객체 중에서) 해당 객체에 도달하기까지 weak reference, phantom reference 없이 soft reference 가 하나라도 존재한다.|
|**weakly reachable**|(strongly reachable, softly reachable 객체가 아닌 객체 중에서) 해당 객체에 도달하기까지 phantom reference 없이 weak reference 가 하나라도 존재한다.|
|**phantomly reachable**|(strongly reachable, softly reachable, weakly reachable 객체 모두 해당되지 않는 객체이다. 이 객체는 finalize 되었지만 아직 메모리가 회수되지 않은 상태이다.|
|**unreachable**|root set으로부터 도달할 수 없는 객체|


### Sweep

(Mark 단계에서 사용되지 않는 메모리로 식별된) 메모리를 해체하는 작업

단, **회수 대상 객체를 처리(finalization)하는 것**과 **메모리를 회수하는 작업**은 다르다는 것을 인지해야 할 것 같다. 

→ Marking 은 후자만을 의미하는 건가?

> STW 를 통해 모든 작업이 중단되고나면, GC는 메모리를 스캔하면서 <br>
> 1. Marking 작업을 한다. <br>
> 2. Sweeping 작업을 한다.

<br>

**Softly Reachable (SoftReference)**

softly reachable 객체는 힙(heap)에 **'남아있는 메모리 크기'** 와 **'해당 객체의 사용 빈도'** 에 따라 GC 여부가 결정된다.

weakly reachable 객체와 달리 GC가 동작할 때마다 회수되지 않는다. 자주 사용될수록 오래 살아남는다.

Oracle HotSpot VM 에서 Softly reachable 객체의 GC를 조절하는 옵션  : `-XX: SoftRefLRUPolicyMSPerMB=<N (기본값 : 1000)>`

<br>

**Weakly Reachable (WeakReference)**

특별한 정책에 의해 GC 여부가 결정되는 `SoftReachable` 과 달리 GC를 수행할 때마다 회수 대상이 된다.

그러나 GC가 실제로 언제 객체를 회수할지는 GC 알고리즘에 따라 모두 다르므로, GC가 수행될 때마다 반드시 메모리까지 회수된다고 보장하지는 않는다. 이는 `soflty reachable` 객체는 물론 `unreachable` 객체도 마찬가지다.

LRU 캐시와 같은 구현을 위해서는 `SoftReference` 보다 `WeakReference` 객체가 유리하다. 

→ 왜지? 최근에 사용되는 객체는 남아있어야 하는거라면 `SoftReference` 도 적절한 거 아닌가?

→ `SoftReference` 는 힙에 남아있는 메모리가 많을수록 회수 가능성이 낮기 때문에, 다른 비즈니스 로직 객체들을 위한 공간들이 `SoftReference` 객체에 점유되는 특징이 있다. <br> 따라서 전체 메모리 사용량이 높아지고 GC가 더 자주 일어나며 GC에 걸리는 시간도 상대적으로 길어지는 문제가 발생한다.

> *" GC가 Marking 하는 작업과 Sweeping 하는 작업은 즉각-연속적으로 발생하는 작업이 아니며, GC 대상 객체의 메모리를 한번에 모두 회수하지도 않는다. "*

<br>

**Phantomly Reachable (PhantomReference) + ReferneceQueue**

https://d2.naver.com/helloworld/329631

<br><br>

## Generation 

HotSpot VM 에서는 물리 공간을 크게 2개로 나누었다. 
- Young Generation
- Old Generation

**Card Table**

- 512 Byte의 Chunk
- Old Generation → Young Generatoin 으로 참조하는 경우에 대해서는 **Card Table** 이라는 것으로 관리한다. <br> Minor GC가 발생할 때 Old 영역을 모두 탐색하는 것을 피하고, **Card Table** 만 확인하도록 하기 위함이다.
- **write barrier** 를 사용하여 관리한다. <br> Minor GC를 빠르게 할 수 있도록 하는 장치이다. <br> 약간의 오버헤드는 발생하지만 전반적인 GC 시간은 줄어들게 된다. <br> (= 바로 위 내용인, old → young 을 참조할 때 card table 에 기록하는 것을 의미한다.)

<br><bR>

## Minor GC

Young 영역에 대해서 GC가 동작한다.

### Young 영역

- Eden 영역
  - 새로 생성된 객체가 할당(Allocation)되는 영역
- Survivor 영역
  - 최소 1번의 Minor GC 에서 살아남은 객체가 존재하는 영역 (→ Eden 영역에서 Survivor 영역으로 옮겨지는 것)
  - Survivor 영역에서 Minor GC 발생 시, 다른 Survivor 영역으로 이동 (→ 즉, 여러 Survivor 영역에 번갈아가며(?) 계속해서 이동)

1. 새로 생성된 객체 → Eden 영역 할당
   - Eden 영역이 꽉 차면 Minor GC 발생
      - 사용되지 않는 메모리는 해제
2. 사용중인 객체 → Survivor 영역으로 옮김
   - Survivor 영역은 2개, 반드시 1곳에만 데이터가 존재해야 함
   - Survivor 2곳 중 1곳은 반드시 빈 영역이 존재 (즉, 꽉 차지 않음)
   - Survivor 한 곳이 꽉 차면 다른 한 Survivor 영역으로 이동
3. 1 ~ 2번 반복 → 계속해서 살아남은 객체는 Old 영역으로 이동 (= Promotion)

<br>

**알아둘 점**

1. Minor GC (Young 영역)에서 살아남은 횟수(age)를 Object Header 에 기록한다. <br> → 이 age 값을 보고 promotion 여부를 결정한다.

2. Survivor 영역 중 1곳은 반드시 사용되어야 한다.<br> → 만약 하나의 데이터가 두 Survivor 영역에 모두 존재하거나, 두 Survivor 의 사용량이 0이라면 비정상적인 상황임을 인지할 수 있다. 

<br>

### Bump the pointer

> *" HotSpot JVM 에서는 Eden 영역에 객체를 빠르게 할당(Allocation)하기 위해 `Bump the pointer`와 `TLABs`라는 기술을 사용하고 있다. "*

**Eden 영역에 마지막으로 할당된 객체의 주소를 캐싱해두는 기술**

- 새로운 객체를 할당할 때, 빈 공간(유효한 메모리 공간)을 탐색해야하는데 이 탐색 비용을 없애줄 수 있는 것이다. <br> (→ 마지막 주소의 다음 주소를 사용하게 한다.)
- 새로운 객체를 할당할 때 객체의 크기가 Eden 영역에 적합한지만 판별하면 된다. <br> (→ 객체의 크기가 크면 어디에 저장될까?)

<br>

### TLABs (Thread-Local Allocation Buffers)

> *" HotSpot JVM 에서는 Eden 영역에 객체를 빠르게 할당(Allocation)하기 위해 `Bump the pointer`와 `TLABs`라는 기술을 사용하고 있다. "*

**각각의 쓰레드마다 Eden 영역에 객체를 할당하는 공간(메모리 Base 주소 처럼?)을 부여함으로써, (멀티쓰레드 환경에서)동기화 작업 없이 빠르게 메모리를 할당하는 기술**

- 각 쓰레드는 자기만의 공간에만 할당한다.
- (싱글스레드 환경은 문제가 없다.) 멀티스레드 환경에서 Eden 영역에 객체를 할당할 때 Lock을 걸어 동기화를 해주어야 한다. <br> → 이때 성능 문제를 해결하기 위해 사용된다.

<br><br>

## Major GC (Full GC)

Old 영역에 대해서 GC가 동작한다.

Minor GC에서 계속 Promotion되어 Old 영역의 메모리가 부족해지면 발생한다.

- (Old 영역에 비해)Young 영역은 크기가 작기 때문에 대게 적은 시간(ex. ~~~ms)으로 수행된다.
- Old 영역은 크며, Young 영역을 참조할 수 있다. → (Minor GC의)~~배 이상의 시간으로 수행된다. 


<br><br>

### 참고

1. https://mangkyu.tistory.com/119
2. https://d2.naver.com/helloworld/1329
3. https://d2.naver.com/helloworld/37111
4. https://d2.naver.com/helloworld/329631