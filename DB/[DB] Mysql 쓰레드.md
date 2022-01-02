**Mysql 서버는 쓰레드 기반으로 동작합니다.**

- **Foreground Thread**
- **Background Thread**

<br>

### Foreground Thread (Client Thread, 사용자 쓰레도)

Client의 커넥션/요청을 처리하기 위해 존재하는 쓰레드입니다. 즉, 최소한 클라이언트가 접속된 만큼 쓰레드가 존재합니다.

Client가 커넥션을 종료하면, Thread Pool로 되돌아갑니다.

Foreground Thread는 데이터를 Mysql의 캐시나 버퍼로부터 데이터를 읽어옵니다. 캐시/버퍼에 데이터가 없으면 디스크로부터 데이터를 읽어옵니다.

**스토리지엔진(InnoDb, MyISAM)에 따라 Foreground Thread가 디스크의 쓰기 역할이 다릅니다.**

- **InnoDB**
  - 디스크 쓰기 작업은 Background Thread 가 처리합니다.
- **MyISAM**
  - 디스크로 쓰기 작업은 Foreground Thread 가 처리합니다. (이유는 읽기 작업을 주로 할 것이기 때문이라고 합니다.)


<br>

### Background Thread

> 스토리지엔진에 따라 Background Thread의 역할이 다릅니다.

**Case1. MyISAM**

Disk 읽기/쓰기 작업이 Foreground Thread 에서 처리되기 때문에 Background Thread 가 크게 할 일이 없습니다.

**Case2. InnoDB**

Disk 쓰기 작업은 Background Thread 가 책임집니다. Disk 쓰기 작업을 위한 (역할이 나뉘어진)여러 쓰레드가 존재합니다. **또, Multi-Thread 방식으로 동작합니다.**

- **Main thread** : 아래의 쓰레드를 관리하는 메인 쓰레드입니다.
- **Log thread** : log 를 disk 에 기록합니다.
- **Write thread** : Buffer의 데이터를 disk에 기록합니다.
- ...
