> http://biscuit.cafe24.com/moniwiki/wiki.php/socket%C7%C1%B7%CE%B1%D7%B7%A1%B9%D6%B1%E2%BA%BB

1. socket()
2. bind()
3. listen()
4. read() / write()


## 1. socket()

socket() 함수는 통신을 위한 하나의 포인트(= endpoint)를 만든다. 통신을 위한 매개체(= fd)를 만든다.

> *" socket() 함수는, 통신을 위한 끝점 한개를 만든다 (무슨 소리냐! :@). 워낙 여러가지 통신에 사용할 수 있는 함수이지만, 이 글에서는 TCP/IP 통신을 위한 부분에 국한해서, 뭔가 통신을 하기위한 매개체(이것을 fd, file descriptor 라고 한다)를 한개 생성하는데 쓴다. "*


```c++
#include <sys/types.h>
#include <sys/socket.h>

int socket(int domain, int type, int protocol);
```
```c++
int fd;

if ((fd = socket(PF_INET, SOCK_STREAM, 0)) < 0) {
    fprintf(stderr, "socket() error\n");
    exit(-1);
}
```

> *" **socket() 함수의 파라미터 domain과 type은 TCP/IP의 IP와 TCP를 의미한다.** 참고로 **UDP통신을 위해서는 type에 SOCK_DGRAM 을 쓴다.** 프로그래밍에서 Stream이라는 단어는 연속적으로 주르륵 붙어있는 자료구조를 가리키는데, 통신을 하다보면 항상 먼저보낸 데이터가 먼저 도착하지는 않는데, **그럼에도 불구하고 데이터의 순서를 TCP라는 통신규약이 바로잡아주기 때문에 Stream이다. 물론 UDP는 그런것이 보장되지 않는다.** "*

socket() 함수는 리턴 값으로 fd 값을 반환한다.

> *" socket()함수는 단지 정수값 한개를 돌려주는데 (**보통은 4이다. 왜냐면 stdin, stdout, stderr 가 각각 1개씩 차지하고 있어서 4번째 fd 가 된다**) 앞으로의 모든 조작은 이 정수값을 통해서 하게 된다 (다른 함수를 호출할때마다 이 값을 넘겨줘야 한다). "*

<br>

## 2. bind()

```c++
#include <sys/types.h>
#include <sys/socket.h>

int bind(int sockfd, struct sockaddr *my_addr, socklen_t addrlen);
```

bind() 함수는 앞서 만든 fd에 소켓의 설정을 연결시켜준다.

> *" bind()함수는 Socket과 Name을 연결시켜(binding) 주는 함수이다(역시 무슨 소리냐! :@). **socket의 성격을 설명하는 sockaddr라는 구조체의 값들을 앞서 만든 fd 값에 연결시켜준다.** "*

```c++
int port_listen = 8000;
struct sockaddr_in server_addr;

...

bzero((char *)&server_addr, sizeof(server_addr));
server_addr.sin_family      = PF_INET;
server_addr.sin_addr.s_addr = INADDR_ANY;
server_addr.sin_port        = htons(port_listen);

if((bind(fd, (struct sockaddr *)&server_addr, sizeof(server_addr))) < 0 ){
    fprintf(stderr, "bind() error\n");
    exit(-1);
}
```

sockaddr_in 을 보면 알 수 있듯이, 소켓에 대한 속성 정보(= 설정)을 위한 구조체다.

위 예제는 `INADDR_ANY(= 어디에서나 접속 가능한)`와 `8000번 포트`를 설정해주고 있다.

> *" sockaddr_in 이라는 구조체가 등장하는데, bind에서 연결할 소켓속성을 담을 구조체이다. 예제에서는 '어디에서나 접속가능한'(INADDR_ANY) 8000 번 포트를 listen 하도록 설정하고 있다. bind() 에서 에러가 나는 경우는 **첫째 이미 다른프로그램이 이 포트를 이용하는 경우, 둘째 당신의 권한이 부족한 경우이다. 슈퍼유저(root)만이 1024번 아래 포트를 쓸 수 있다.** "*

<br>

## 3. listen()

소켓에 대한 생성,설정 이후 클라이언트의 요청을 받기 위한 명령어다.

> *" listen 함수는, 이제 클라이언트들을 위해 귀를 기울여라!하고 최종명령을 내리는 것이다. 여태까지를 정리하면, **(1) socket을 생성 (2) sockaddr_in 구조체를 만들어 대충 채우고 (3) bind() 로 1과2 두개를 묶고, (4) listen 하는 순서**이다. 이렇게 해서 **서버는 연결을 받아들일 준비가 된다. 실제로 클라이언트가 연결을 하면 서버는 accept 함수로 연결을 받아들이고, 적당한 때에 해당 소켓에서 데이터를 읽고 또 적당한 때에 쓰면 되는 것.** "*
> 
> *" 바로 이 **적당한 때를 구현하는 방법이 사실 소켓 프로그래밍의 핵심**이다!! "*

<br.

## 4. read() / write()

```c++
#include <unistd.h>

ssize_t read(int fd, void *buf, size_t count);
ssize_t write(int fd, const void *buf, size_t count);
```

소켓의 데이터를 읽거나, 소켓에 데이터를 쓸 때 사용한다. (= 파일에 데이터를 읽고, 쓰는 것과 동일하다.)

> *" read(), write()는 사실 소켓과는 별로 상관이 없지만, **unix의 모든 장치는 FILE이다**라는 철학에 의해 소켓에 데이터를 쓰거나 읽을때 사용한다. 파일에 쓰거나 읽는거와 다르지 않다. 다만, 소켓연결이 끊기거나 에러가 발생했을 경우 부수적인 처리들이 필요하다. "*
> 
> 소켓은 별 게 아니고, 파일이다. :star:

<br>

```c++
#define MAX_BUF 1024

int nread;
char buf[MAX_BUF];

nread = read(fd, buf, MAX_BUF);

if(nread <= 0) {
    if(nread < 0){
        // some error handling routine
    }
    close(fd);
}
```

fd가 가리키는 곳으로부터 MAX_BUF 만큼 데이터를 읽는다. read()의 리턴 값이 0인 경우 EOF 를 의미하고, -1인 경우 에러를 의미한다. (0, -1일 때 소켓 종료 처리에 신경써야 한다.)

write()도 read()와 유사하다.

> *" write()의 경우는 좀더 신경써야 할 일이 많다. 그 중 가장크게 신경을 써야 하는 것은 WOULDBLOCK이다. **운영체제 내부적으로 통신을 위한 버퍼를 가지고 있는데, 이 버퍼보다 큰 데이터를 보내려고 하거나 이미 가득 차 있는데 무언가를 더 보내려고 하는 경우에 WOULDBLOCK이 발생하게 된다.** ... **이런일은 일시적인 통신지체 (흔히 '랙') 상황에서 일반적으로 발생하는데 그냥 연결을 끊어버릴 수도 없고, 버퍼가 비워질때까지 무작정 기다릴 수도 없는 노릇. 그래서 Send Buffering 기법을 쓴다.** **보내야 할 데이터가 발생하면 바로 전송하지 않고 어플리케이션 버퍼에 넣어두며, 적당한 시점에서 실제 전송을 시도하도록 하는 방법**이다. "*