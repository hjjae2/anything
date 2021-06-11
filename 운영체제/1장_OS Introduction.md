### OS Introduction

**OS 란?**

1. 프로그램이 동작하는 것을 쉽게 해준다.
2. 시스템을 효율적으로, 정확하게 운영한다.
3. **'자원관리자' 이다.**
   - 물리 자원 : CPU, Dram(Memory), Disk, keyboard(KBD), Network, ...
   - 가상 자원 : Process, Thread, Virtual Memory, Page, File, Directory, Driver, Protocol, ...

**\* OSTEP(OS Three Easy Picese) : Virtualization, Concurrency, Persistence**

**\* OS 의 기본 목표 : Abstraction, Performance, Protection, Reliability, ...**

<br>

**컴퓨터 시스템의 계층**

```
user1   user2   user3   ...   userN

 --------------------------------------
| System Program & Application Program |
|      OS                              |
 --------------------------------------
 --------------------------------------
|                  HW                  |
 --------------------------------------
```

<br>

프로그램을 수행하면 아래와 같은 것들이 진행된다.
- loading
- memory management
- file management
- scheduling
- context switch
- i/o processing

<br>


|키워드|내용|
|-|-|
|Memory|Address + Content(instruction + data)|
|Register|CPU 내부에 있는 data holding elements|
|Interept|HW 가 kernel 에 event 의 발생을 알리는 것|
|Trap|SW 가 kernel 에 event 의 발생을 알리는 것|
|Scheduling|다음에 수행할 프로세스를 어떻게 선택할 지 결정하는 것<br>자원을 누구에게 줄지 정하는 것|
|IPC|Inter Process Comunication<br>Process 간 통신하는 것|
|Signal|Process 에게 event 를 알리는 것(단위(?))|
|Mode|Kernel mode<br>User mode<br>ex. system call 을 사용하기 위해 mode switch 가 발생한다.|
|System call|OS 가 제공하는 API(인터페이스)|

<br>

\* 명령어 수행 시 : fetch -> execute (fetch : memory -> IR)

<br>

**CPU 가상화 (CPU Virtualization)**

각각의 Process 는 자기만의 CPU 를 갖는 것 처럼 동작한다.

CPU 의 수가 적어도 가상화를 통해 많은 수의 Process 를 실행할 수 있게 한다.

<br>

**Memory 가상화 (CPU Virtualization)**

각각의 Process 는 자기만의 Memory 를 갖는 것 처럼 동작한다.

Process 사이에서 memory 의 독립성을 보장한다. 

<br>

**병행성 (Concurrency)**

여러 Process 가 동시에 실행되는 것 처럼 보이는 것 (혹은 그렇게 느낄 수 있도록 해주는 것)

> 병행성(Concurrency) vs 병렬성(Parallelism)https://nesoy.github.io/articles/2018-09/OS-Concurrency-Parallelism

<br>

**영속성 (Persistence)**

자원(Data)을 영구히 보존하는 것 (file)

\* File System 의 오류 핸들링 : journaling, cow(copy-on-write)

\* File System 의 성능 향상법 : cache, delayed write