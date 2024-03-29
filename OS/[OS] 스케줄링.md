## 스케줄링

스케줄링이란, 시스템의 자원을 어떤 프로세스에게 할당할 것인지 선택하는 것이다.

선점형 스케줄링, 비선점형 스케줄링이 있다.

<br>

### 선점형 스케줄링

어떤 프로세스가 시스템의 자원을 사용하여 작업을 진행하는 도중, 해당 작업을 중지하고 시스템의 자원을 다른 프로세스에게 넘겨줄 수 있는 스케줄링이다.

**Round Robin (RR)**

FCFS 기법 + 선점형의 기법이다.

<br>

**SRT (Shortest Remaining Time)**

SJF + 선점형의 기법이다.

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

> Reference
> 1. https://velog.io/@codemcd/%EC%9A%B4%EC%98%81%EC%B2%B4%EC%A0%9COS-6.-CPU-%EC%8A%A4%EC%BC%80%EC%A4%84%EB%A7%81