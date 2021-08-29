### Concurrency : Thread and Lock

> \* 병행성 : 동시에 처리되는 것 처럼 보이게 하는 것
> 
> \* 병렬성 : 동시에 처리되는 것

Keywords of concurrency

- Shared data
- Race condition
- Coarse-grained locking
- ...

Fron now on ...
1. Multi-threaded Program : multiple control flows in a program
2. Concurrency : shared data -> race condition

<br>

**What is a Thread ?**

Computing resources for a program

Thread는 프로세스 내에서 실행되는 **흐름의 단위**를 말한다.

Address space 에서 code(text), data, heap 영역을 공유한다.
- fast creation
- better sharing
- worse isolation

<br>

**Benefit of thread**

1. Fast creation
2. Parallelism
    - Multithread : devide & conquer (MapReduce model)
3. Can overlap processing with waiting
    - Ex. Webserver : request thread, processing thread, response thread, ...
4. Data sharing

<br>

**Issue of multithread**

Uncontrolled scheduling

1. Atomicity : do all or none
2. Critical section : race condition problem
3. Mutual exclusion : allow only one thread to access the critical section

<br>

**Issue of concurrency**

1. Mutual exclusion
2. Synchronization

<br>

**Evaluation of locking(unlocking)**

1. Correction : Does it work correctly?
2. Coarse-grained lock : lock 범위 크게 (단순, 낮은 효율)
3. Fine-grained lock : lock 범위 작게 (복잡, 높은 효율)

<br>

**How to build the locking(unlocking)**

1. Disable interrupt
   - single cpu 에서는 문제가 없다.
   - multi cpu 에서 문제가 있다.
   - 남용/오용의 가능성이 있다.
2. SW
   - Dekker's algorithm, Peterson's algorithm, ...
   - 잘 사용되지 않는다. (이해하기 어렵고, 비효율적이다. 가끔 부정확하다.)
3. HW (HW atomic operation)
   - **Test-and-Set instruction**
   - **Compare-and-Swap**
   - **Load-Linked and Store-Conditional**
   - **Fetch-and-Add**
   - ...

<br>

**Lock Mechanisms**

1. Spin Lock 
   - sleep lock 에 비해 비효율적이다.
2. Sleep Lock
   - context switch overhead 발생한다.

\* context switch 의 비용이 spin lock 보다 비효율적이라면, spin lock 을 사용한다. **(짧게 기다릴 때 : spin lock, 오래 기다릴 때 : sleep lock)**