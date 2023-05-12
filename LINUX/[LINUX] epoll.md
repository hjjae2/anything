> http://blueheartscabin.blogspot.com/2013/08/c-epoll.html

## [select, poll, epoll](http://biscuit.cafe24.com/moniwiki/wiki.php/epoll#s-5)

관심 있는 fd (= 대상 fd)들을 등록해두고 이들 중 이벤트가 발생하는 것을 감지하기 위해 사용하는 함수다.

다만, 이벤트를 감지하는 동작 방식에 차이가 있다.

select : 어떤 fd에 발생한 이벤트인지 찾기 위해 등록된 fd 리스트를 선형 탐색한다.

epoll : 이벤트가 발생한 fd들을 반환해준다.

> 이해한 내용이 맞는 지 다른 글도 확인

<br>

## [epoll 프로그래밍 흐름](http://biscuit.cafe24.com/moniwiki/wiki.php/epoll#s-6)

1. 소켓 생성, 설정
   1. socket(), bind(), setsockopt()
2. epoll 에 등록 (`epoll_ctl`)
3. `listen()`
4. `epoll_wait()`
   1. epoll_wait 를 통해 이벤트 발생을 감지/accept : accpet() :thinking:
5. accpet 로 부터 넘어온 fd와 통신 준비
   1. `fcntl()`
6. epoll 에 새로운 fd 등록 : epoll_ctl

<br>

### epoll_ctl

**epoll이 관찰할 fd, 이벤트를 등록하기 위한 인터페이스**라고 한다.

> *" epoll_ctl은 epoll이 관심을 가져주길 바라는 fd와 그 fd에서 발생하는 관심있는 사건의 종류를 등록하는 인터페이스 "*

```c++
#include <sys/epoll.h>

int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
```

<br>

### epoll_wait

```c++
#include <sys/epoll.h>

int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
```

epoll 의 핵심 인터페이스.

epoll_wait 은 등록한 fd에 이벤트가 발생하는 것을 확인한다. 

- `(epoll_event).events[]` :  epoll의 이벤트 감지 결과는 select, poll 과 달리 `(epoll_event).events[]` 의 배열로 전달한다. 
- `maxevents` : 최대 처리할 event 수를 지정할 수 있다.
- `timeout` : 해당 시간만큼 기다린다.


> *" **간단한 채팅서버의 경우를 살펴보자.** 서버가 어떠한 일을 해야하는 시점은 이용자 누군가가 데이터를 보내왔을 때인데, 아무도 아무말도 하지 않는다면 서버는 굳이 프로세싱을 할 이유가 없다. 이럴때 timeout을 (-1)로 지정해두고 이용자들의 입력이 없는 동안 운영체제에 프로세싱 타임을 넘기도록 한다.*
> 
> ***온라인게임(특히 MMORPG)의 경우**에는, 이용자의 입력이 전혀 없는 도중이라고 하더라도, 몬스터에 관련된 처리, 적절한 저장, 다른 서버와의 통신들을 해야 하므로 적절한 timeout (필자의 경우에는 1/100 sec, 즉 10ms를 선호한다)을 지정해 주도록 한다.*
> 
> ***뭔가의 프로세싱을 주로 하면서 잠깐잠깐 통신이벤트를 처리하고자 하는 경우**, 즉 프로세스의 CPU 점유를 높게해서 무언가를 하고 싶은 경우에는, timeout 0을 설정하여 CPU를 독점하도록 설계할 수도 있다.*
> 
> ***별도 thread를 구성하여 이 thread 가 입출력을 전담하도록 프로그램을 작성하고자 하는 경우에는**, 당연히 timeout을 (-1)로 설정하여 남는 시간을 다른 thread, 혹은 운영체제에 돌려 주도록 한다. "*

```c++
#define MAX_EVENTS 100

struct epoll_event events[MAX_EVENTS];

for(;;) {
    // 이벤트가 발생한 fd의 개수가 반환된다.
    int nfds = epoll_wait(fd_epoll, events, MAX_EVENTS, 10);

    if(nfds < 0) {
        // ERROR
        ...
        exit(-1);
    }
    if(nfds == 0) {
        // IDLE (아무 이벤트도 발생하지 않음)
        continue;
    }

    // 이벤트가 발생함
    for(int n=0; n<nfds; n++) {
        // 이벤트 처리
        OnEvent(&events[n]);
    }
}
```