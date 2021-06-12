### Memory Management

**Memory Virtualization Keywords**

1. Virtual address
2. (Private) Address space
3. HW help

<br>

**Address Space 개념의 변천사**

Early System -> Multiprogramming & Time sharing -> Virtual memory(Address space)

<br><br>

**Ealry System**

Single programming system

물리 메모리를 직접적으로 사용한다.

물리 메모리보다 큰 메모리가 필요하다면, **Overlay** 기법을 사용한다.

\* Overlay : 현재 꼭 필요한 Part 만 메모리에 올리는 것 (Part 번갈아가면서 실행)

<br><br>

**Multiprogramming & Time sharing**

> 컴퓨터 HW 가 좋아지면서, 한번에 여러 개의 프로그램을 올릴 수 있게 되었다 (?)

Multiprogramming : Multiple processes are ready to run

Time sharing : Switch CPU among ready processes

이슈
- Protection
- Free space (It isn't easy to find free space)

<br><br>

**Virtual memory (Address space)**

위의 이슈(protection, free space) 를 해결한다.

**\* Address space : code(text), data, heap, stack**

각각의 프로세스는 자신들의 고유한 Address space(Virtual memory) 를 갖는다.

이 Virtual memory 는 결국 물리 메모리에 올려져 실행될 것이다.

목표
- Transparency (투명성) : 프로그래머가 메모리의 사이즈, 사용가능한 공간에 대해 알 필요가 없다.
- Eficiency (효율성, 성능) : 시간/공간적 효율성을 보장한다
- Protection, Isolation (보호, 독립성) : 프로세스(혹은 그것의 메모리)를 보호한다.

code(text), data 영역 : 할당받는 사이즈가 고정적이다.<br>
stack, heap 영역 : 할당받는 사이즈가 동적이다. (stack : 컴파일러에 의해, heap : 개발자에 의해)

**메모리 사용을 잘못 했을 때, 주로 Segmantation fault 가 발생한다.** 

<br><br>

**Address Translation**

가상 메모리주소와 물리 메모리주소를 변환(매핑)시켜줄 수 있어야한다.

```
가상 메모리의 시작은 0 부터 시작하고, 물리 메모리의 100 위치에 올려졌다고 가정한다.

가상 메모리의 10 의 주소를 읽을 때, 물리 메모리의 110 위치를 읽을 수 있어야 한다.
```

> **Hardware support**
>
> **MMU (Memory Management Unit)**
>
> Address translation 을 돕는 CPU 의 한 part 이다.
>
> - Base/Limit registers, Segmentation related registers, Paging related registers, TLB + Circuitry

<br>

### Memory Management

- Base / Limit Registers
- Segmentation
- Paiging

<br>

**Base & Limit registers**

> Other mechanisms : Segmentation, Paging

Base ~ Base + Limit 구간을 사용한다. (Base 보다 작거나, Base + Limit 보다 큰 곳을 사용하면 Segmant fault 발생)

```
Base register : 32KB
PC : 128

물리 메모리 주소 : 32768 + 128 = 32896
```

<br>

**Segmentation**

Base & Limit Register 메커니즘의 경우는 필요한 메모리가 통으로(연속해서) free 해야 물리 메모리에 올릴 수 있다. 이 문제를 해결하기 위한 메커니즘 중 하나이다.

Base & Limit register 기법을 base 로 한다.

프로세스를 논리적으로 나누고 이 각각의 영역을 'segmant' 라고 한다. (예: code, data, stack, heap 영역으로 나눈다. 혹은 이것을 더 세분화 한다.)

이 세그먼트들은 분리(독립)하여 물리 메모리에 올린다. (이때 각각의 segment 는 각각의 base & limit register 를 갖는다.)

> \* Segment table 을 사용해 address translation 을 진행한다.

- 물리 메모리의 주소 : Segment number + offset 

- Sharing : 특정 segment 영역을 공유할 수 있다.

- Protection : 세그먼트 별로 proteciton 정책을 정할 수 있다. (granularity)

**Segment-Size**

- Coarse-grained (크게 작업) : 관리 쉬워진다.

- Fine-grained (작게 작업) : 효율성 향상, 관리를 위해 segment table 사용한다.

**Allocation**

- Best-fit
- Worst-fit
- First-fit
- Buddy algorithm

문제점

- 외부 단편화 : segment 의 사이즈는 가변적인데, 이것은 외부 단편화를 특히 더  야기시킨다.

<br>

### Paging

Page table 사용 (Page number, offset : page size * page number + offset)
 - 32bit 환경에서 page 의 크기 : 4KB (보통)
 - Memory 에 저장 (CPU register 에 저장하기엔 사이즈가 크다.)

Fixed size 사용 (관리 용이해진다 -> HW 적으로도 support 원활해진다.)


> References
> 1. Virtual Memory, https://velog.io/@woo0_hooo/OS-CH3-3-Virtual-Memory
여기까지는 paging 의 속도는 굉장히 느리다.
- PTE 주소를 찾아야한다. (memory)
- PTE에서 fetch 할 때에도 memory 에 접근한다.
- Bits 를 체크한다. (PTE Bits)
- 물리 메모리에 올릴 때, 또 memory 에서 읽는다.

\* **TLB(Translation Lookaside Buffer) 기법 -> Faster translation**

- Cache of recent used PTE (Better name would be an address-translation cache)
- **\* 가능한 cache 를 사용하자!!**
