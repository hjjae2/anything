### Scheduling

**스케줄링** 이란, 시스템의 자원을 어떤 프로세스에게 할당할 것인지 선택하는 것이다.

누가(어떤 프로세스) 자원(CPU)을 사용할지 선택하는 것이다.

선점형 스케줄링, 비선점형 스케줄링이 있다.

> \* Workload : 일의 양

<br>

**Scheduling Metric**

1. 완료 시간 (Turnaround time)<br>
- Turnaround time = Completion time - Arrival time
- Batching System 에 좋은 metric 이다.

2. 응답 시간 (Response time)<br>
- Response time = First run time - Arrival time
- I/O 에 좋은 metric 이다. (사용자는 빠른 응답시간이 중요하다.)


3. 공평성 (Fairness)<br>

4. 처리율 (Throughput)<br>
   
5. 임계시간 (Deadline)<br>

\* 설계 목적에 따라 우선시 되는 기준(metric)이 달라진다.

\* 모든 것을 만족시킬 수는 없다.

<br>

### 비선점형 스케줄링

어떤 프로세스가 시스템의 자원을 사용하여 작업을 진행하게 되면, 종료될 때까지 자원을 뺏을 수 없다. 기다려야 한다.

**FCFS (FIFO)**

선입 선출의 스케줄링 기법이다.

<br>

**SJF (Shortest Job First)**

실행시간(처리시간)이 짧은 것부터 자원을 점유한다.

<br>

**HRN (Hightest Response-ratio Next)**

SJF의 Starvation (기아, 굶주림) 을 해결하기 위한 기법이다.

**우선순위 = 실행시간 + 대기시간 / 실행시간** 의 계산을 통해 우선순위를 정한다.

<br>

### 선점형 스케줄링

어떤 프로세스가 시스템의 자원을 사용하여 작업을 진행하는 도중, 해당 작업을 중지하고 시스템의 자원을 다른 프로세스에게 넘겨줄 수 있는 스케줄링이다.

**Round Robin (RR)**

FCFS 기법 + 선점형의 기법이다.

Time slice 를 설정하여 해당 time 동안만 실행되고 자원을 넘긴다.

Time slice 가 짧다면, 응답률은 좋지만 높은 context switch overhead 가 발생한다.

Time slice 가 길다면, 응답률은 안좋지만 낮은 context switch overhead 가 발생한다.

<br>

**SRT (Shortest Remaining Time, STCF, Shortest Time to Completion First)**

SJF + 선점형의 기법이다. 

남은시간(완료 되기까지의 시간)이 짧은 것을 우선적으로 선택한다.

<br>

**MLQ**

> Queue 에 자원을 점유할 프로세스들을 대기시킨다.

여러 개의 Queue 를 갖는다. Queue 마다 우선순위가 있다.

우선순위가 높은 Queue 에 대기하는 프로세스가 자원을 점유한다. 

선점형 기법이기 때문에, 우선순위가 높은 프로세스가 대기 상태로 들어오면 바로 자원을 점유할 수 있다. (사용중이던 애는 자원을 빼앗긴다.)

**\* MLQ 를 이용했을 때의 특징은 I/O 와 같이 처리 시간이 빨라야 하는 것은 우선순위가 높은 Queue에 대기시키고, 비교적 처리 시간이 느슨해도 되는 프로세스는 우선순위가 낮은 Queue 에 대기시킬 수 있다는 것이다.**

<br>

**MFQ**

MLQ + Time Slice, 큐 간의 이동 등의 기법이 추가되었다.

MLQ 와 동일하게 Queue 에 우선순위가 있다. 그러나 Queue 마다 Time slice 가 있으며, 해당 Time slice 동안 종료되지 않는 프로세스는 다음 Queue(우선순위가 낮은)로 이동된다. 

Queue 에서 일정한 대기시간이 지나면 우선순위가 높은 Queue 로 이동될 수 있다.

이슈
- How many queues?
- How big should the time slice be per queue?
- How often do the periodic boost?
- ...

\* 보통 우선순위가 낮은 큐일수록 time quantum(time slice) 가 길다.

\* 하지만 Starvation 을 완전히 방지할 수는 없다. -> **Periodic Boosting**

\* **Periodic Boosting** : (주기적으로) 모든 작업을 Top Queue 로 올린다.

<br>

(+) 

### Proportional Share Scheduler (Fair Share Scheduler)

위에서 소개한 스케줄링 기법들은 Turnaround time, Response time 들의 최적화를 위한 스케줄링 기법이다.

그러나 이 스케줄링은 각각의 프로세스에게 CPU 의 사용량(사용 시간)을 균등하게 배분하는 것에 목적이 있다.

**Lottery Scheduling**

복권에 당첨된(즉, 랜덤에서 당첨된) 프로세스가 자원을 사용한다.

이때 Ticket 개념(the share of a resource)이 사용된다. 더 많은 ticket 을 가질수록 복권에 당첨될 확률이 높아진다.

> Ticket machanisms
> 
> 1. Ticket currency : Allow users to allocate tickets
> 2. Ticket transfer : A job can hand off its tickets to another job (client/server 환경에서 유용하다.)
> 3. Ticket inflation : Temporarily raise or lower the # of tickets
> 
> 특징
> 
> 1. 구현이 간단하다. (random, counter, ticket, ...)
> 2. Non-deterministic
> 3. 어떤 프로세스에게 티켓을 많이/적게 할당해야할지 고민해야한다.
> 4. 프로세스별로 티켓이 할당되더라도, 랜덤(Non-deterministc)한 성격이기에, 보장할 수 없다.

**Stride Scheduling**

A deterministic fair share scheduler

**\* Stride : inverse in proportion to the # of tickets (= ticket / N)**

각각의 프로세스는 Stride, Pass value 값을 갖고 있는다.

Pass value 가 가장 작은 프로세스를 선택한다.

프로세스가 실행될 때 마다, pass value 에 stride 를 더해준다.

다만, 중간에 새로운 Process(pass value : 0) 가 들어온다면, 해당 프로세스가 일정시간 동안 자원을 계속해서 차지할 수 있다.

<br>

### Multiprocessor Scheduling

Parallel program

Scheduler that can handle multiple CPUs

<br>

**CPU Cache(L1, L2, LLC)**

- Cache hit

- Delayed write

Temporal locality (시간 지역성) : 한 데이터가 사용이 되면, 그 데이터는 곧 다시 사용될 가능성이 높다.

Spatial locality (공간 지역성) : 한 데이터가 사용이 되면, 그 데이터 주변에 있는 것들도 사용될 가능성이 높다.

Cache 는 Multiprocessor 에서 굉장히 복잡하게 동작한다.

아래와 같은 상황에서 incoherence 문제가 발생한다.
```
CPU1 에서 A 라는 데이터를 읽었고, 수정했다 (A -> A')
(이때 A' 값을 바로 write 하지 않고, Delayed write 한다고 가정한다.)

이때 CPU2 에서 A 라는 데이터를 읽었다. (A' 를 읽지 못한다.)
``` 

Bus Snooping : A mechanism for supporting coherence
- Monitoring cache, invalidate or update if data is modified

<br>

**Issue on Multiprocessor**

Synchronization

Cache affinity

...

<br>

**Scheduling for Multiprocessor**

**SQMS (Single Queue Multiprocessor Scheduling)**

한 큐를 여러 CPU 가 사용

장점 : 간단

단점 : Cache affinity(캐시 선호도) 낮음, Scalability(확장성) 낮음

**MQMS (Multi Queue Multiprocessor Scheduling)**

SQMS 의 반대 ( + 자원 경쟁 하락 )

<br>
<br>

> Reference
> 1. https://velog.io/@codemcd/%EC%9A%B4%EC%98%81%EC%B2%B4%EC%A0%9COS-6.-CPU-%EC%8A%A4%EC%BC%80%EC%A4%84%EB%A7%81
> 2. Fair Share scheduling, https://icksw.tistory.com/125