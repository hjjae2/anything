# Garbage Collection 종류


## Serial GC (-XX:+UseSerialGC)

- Marking → Sweeping → Compaction
- 싱글 코어(쓰레드)를 위한 GC
- 절대 사용하지 않는 것을 권장

<br><br>

## Parallel GC (Throughput GC) (-XX:+UseParallelGC)

- Serial GC 알고리즘 동일
- Serial GC 알고리즘 + 멀티 쓰레드
- 메모리가 충분, 코어 수가 많을 때 유리

![](../images/[JAVA]%20Garbage%20Collection%20(종류)_56.png)

<br><br>

## Parallel Old GC (-XX:+UseParallelOldGC)

- Marking → Summary → Compaction
- Parallel GC와 비교하여 Old 영역의 GC 알고리즘만 다르다.
- Summray 단게는 앞서 GC를 수행한 영역에 대해서 별도로 살아있는 객체를 식별한다? 약간 더 복잡하다. (찾아볼 것)

<br><br>

## CMS GC (Low Latency GC) (-XX:+UseConcMarkSweepGC)

- Marking(Initial Mark, Concurruent Mark, Remark) → Concurrent Sweeping
- STW 시간이 매우 짧다.
- 모든 애플리케이션의 응답 속도가 매우 중요할 때 사용할 수 있다. (= `Low Latency GC`)
- 다른 GC 방식보다 자원(메모리/CPU)를 더 많이 사용한다. (단점)
- Compaction 단계가 기본적으로 없다. (단점)
  - 조각난 메모리가 많아져 이후 Compaction 작업을 실행하면 다른 GC STW 시간보다 (STW 시간이) 더 길어질 수 있다. (얼마나 자주, 오랫동안 수행되는지 꼭 확인이 필요하다.)

![](../images/[JAVA]%20Garbage%20Collection%20(종류)_39.png)

**Initial Mark**

- 클래스 로더에서 가장 가까운 객체 중 살아있는 객체만 찾는 것으로 끝낸다. (= 매우 짧은 시간)

**Concurruent Mark**

- **Initial Mark** 단게예서 살아있다고 확인한 객체에 대해서 참조를 따라가면서 확인한다.
- 다른 쓰레드가 실행 중인 상태에서 동시에 실행된다. (**concurrent**)

**Remark**

- **Concurrent Mark** 단계에서 새로 추가되거나 참조가 끊긴 객체를 확인한다. (= Concurruent 에 의해 발생할 수 있는 변경 사항을 확인한다.)

**Concurrent Sweep**

- Concurrent Sweeping 을 진행한다.

<br><br>

## G1 GC

> CMS GC를 대체하기 위해 만들어졌다고 한다.

> jdk11 부터 기본 GC 알고리즘으로 적용되었다.

> HW 발전 → 대용량 메모리에 적합한 솔루션을 제공하기 위해 고안되었다.

- Region 개념 도입
  - Eden, Survivor, Old 영역이 존재하지만, 고정된 크기가 아니며 전체 Heap 메모리 영역을 Region 이라는 특정한 크기로 나눴다.
  - Region 은 상태에 따라 역할(Eden, Survivor, Old)이 동적으로 변경된다.
  - Default size = 전체 Heap 메모리 / 2048 

<img src="../images/[JAVA]%20Garbage%20Collection%20(종류)_51.png" width="60%">

<br>

**Humonogous**

(Region 크기의 50%를 초과하는) 큰 객체를 저장하기 위한 공간

<br>

**Available/Unused**

아직 사용되지 않은(사용 가능한) Region

<br>



<br><br>

## ZGC

- 

<br><br>

### 유의 사항

> *" 어떤 서비스에서 A라는 GC 옵션을 적용해서 잘 동작한다고 그 GC 옵션이 다른 서비스에서도 훌륭하게 적용되어 최적의 효과를 볼 수 있다고 생각하지 말라는 것이다. "*


> *" 각 서비스의 WAS에서 생성하는 객체의 크기와 생존 주기가 모두 다르고, 장비의 종류도 다양하다. WAS의 스레드 개수와 장비당 WAS 인스턴스 개수, GC 옵션 등은 지속적인 튜닝과 모니터링을 통해서 해당 서비스에 가장 적합한 값을 찾아야 한다. "*

<br><br>

### 참고

1. https://mangkyu.tistory.com/119
2. https://d2.naver.com/helloworld/1329
3. https://d2.naver.com/helloworld/37111
4. https://d2.naver.com/helloworld/329631