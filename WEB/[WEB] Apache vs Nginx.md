## Apache

Apache 는 MPM(Multi Process Module) 방식으로 동작한다.

MPM 은 다시 아래와 같은 방식이 있다.

- Prefork MPM
- Worker MPM
- Event MPM

<br>

### Prefork MPM

각각의 요청을 각각의 프로세스(1 therad) 가 처리한다.

프로세스 별로 처리하기 때문에 안정적이다. (= 메모리 영역을 공유하지 않는다, 메모리 공간이 독립적이다)

자원의 사용량이 크다.

default 개수만큼 apache 자식 프로세스를 생성해놓는다. 프로세스는 client 요청을 처리하고, 요청이 많을 경우 Process를 새로 생성하여 처리한다.

<br>

### Worker MPM

Prefork와 같이 Default로 Apache 자식 프로세스를 생성해놓는다. 요청이 많아지면 프로세스가 아닌 **각 프로세스의 Thread**를 생성해 처리하는 구조이다. (두 방식의 특징은 우리가 흔히 알고 있는 프로세스와 쓰레드 사용의 장/단점과 동일하다.)

프로세스당 여러 개의 Therad가 존재한다.

각각의 요청을 각각의 스레드가 처리한다. (최대 64개의 Therad 처리가 가능하다. (?))

각각의 Therad 는 한번에 한 connection 을 담당한다.

통신량이 많은 (대규모) 환경에 적합하다.

자원의 사용량이 Prefork 방식에 비해 적다.

Race Condition 이 발생할 수 있다.

<br>

### Event MPM

> Prefork, Worker 방식은 1개의 process(or therad)가 클라이언트와 연결된다. 연결이 끝나지 않는 한 process(or thread)가 소멸되지 않는다. (Keep Alive) 따라서, 대량 접속 환경에서 성능이 급격히 떨어졌다. 이러한 문제를 해결하기 위해 Apache 2.4부터는 Event MPM을 사용할 수 있게 되었다.

(Prefork, Worker 방식의 단점을 보완하기 위해) Apache 2.4 버전에서 릴리즈 되었다.

Worker 방식을 기반으로 한다.

클라이언트로 데이터를 기다리도록 유지되는 단점을 보완하기 위해 각 프로세스에 대한 전용 리스너 스레드를 사용하여 Listening 소켓, Keep Alive 상태에있는 모든 소켓, 처리기 및 프로토콜 필터가 작업을 수행 한 소켓 및 나머지 유일한 소켓을 처리한다. (?)

<br>

## Nginx

시간이 지남에 따라, 동시 접속 / 대량의 트래픽이 발생했고 Apache 처리 방식(MPM)의 단점으로 인해 Nginx 가 탄생했다.

Nginx 의 처리 방식은 `비동기 event-dirven` 방식이다.

한 개 또는 고정된 프로세스만을 생성한다. (= 프로세스가 한 개라면 context switching 으로 인한 오버헤드가 발생하지 않는다.) 이런 프로세스 내부에서 비동기방식을 통해 요청을 처리한다. (동시 접속 요청이 많아도 Process, Thread 생성 비용이 들지 않는다.)

> Event-Driven 방식에선 작업을 하다 I/O, socket read/write 등 CPU가 관여하지 않는 작업이 시작되면 기다리지 않고 바로 다른 작업을 수행한다. 진행중인 I/O 등의 작업들이 끝나면 아! 내가 아까 했던 작업을 다시 진행하면 된다는 이벤트가 발생하고 그 작업을 처리하게 된다.


`이벤트를 받는 reactor`, `실제 처리를 위한 worker`, `worker 로 전달하기 위한 handler`  등으로 구성된다.

> 다만, 긴 많은 I/O처리가 필요한 작업의 경우 시스템 큐에 요청이 쌓이게되어 성능이 저하 될 수 있다.

> 복잡한 처리나 대용량 데이터처리가 필요한 서비스 등에서는 적합하지 않을수 있다.


<br><br>

> Reference
> 1. [Nginx & Apache 비교](https://velog.io/@ksso730/Nginx-Apache-%EB%B9%84%EA%B5%90)
> 2. [Apache MPM 이란?](https://s-jg.tistory.com/27)
> 3. [Nginx와 Apache의 구동방식 비교](https://taes-k.github.io/2019/03/08/server-nginx-event-driven/)
> 4. [넌 뭐니 NGINX?](https://medium.com/sjk5766/%EB%84%8C-%EB%AD%90%EB%8B%88-nginx-9a8cae25e964)
> 5. [Nginx 개념의 이해](https://kbs4674.tistory.com/126)