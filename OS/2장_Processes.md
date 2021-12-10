### Processes

\* Program : Disk 상의 file

\* Process : Program 실행 상태

**Process 를 운영하기 위해 필요한 것**
1. CPU : registers (IR, PC, SP, ...)
2. Memory : Address Space (stack, heap, data, text)
3. I/O information : Opened files/devices

<br>

**Multiple Processes 운영 방식**

Time sharing 기법 사용

- context switch 발생

<br>

**Program 실행 시 OS 의 역할**

1. Load (적재)
   - Disk -> Memory
   - Eagerly vs Lazily
2. Dynamic Allocation (동적 할당)
   - stack 
   - heap
   - initialize parameters (argc, argv)
3. Initialization (초기화)
   - file descriptors(0, 1, 2)
   - I/O
4. Jump to main (main 함수 실행)

<br>

**Process 의 Lify-Cycle (상태)**

```
new(생성)                                                              terminated(종료)
     <------>                                                  <------>
              ready(준비) <------------------> running(동작/운영)
                   <------>              <------>
                            waiting(대기)
```

<br>

**PCB (Process Control Block, Task Structure)**

Process 를 관리하기 위한 자료구조

Process 생성 만들어진다.

```
 -------------
|Process State |
 -------------
|Process Number|
 -------------
|      PC      |
 -------------
|  Registers   |
 -------------
|      ...     |
 -------------
```

<br>

**fork()**

Create a new process : 새로운 프로세스를 생성한다. (Parent - Child 관계)

Return two values : 2개의 값을 반환한다. (Parent, Child)

Non-determinism: 어떤 것이 먼저 실행될지 모른다.

<br>

**wait()**

자식 프로세스가 종료될 때 까지 기다린다. (Synchronization)

<br>

**exec()**

memory image 를 새 것으로 바꾼다.

> exec() 밑의 코드는 실행되지 않는다.

> Modular approach of unix(especially for shell): fork -> exec (on child)

**그 외**

**getpid(), kill(), signal(), ...**