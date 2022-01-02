메모리 구조는 아래와 같이 구분됩니다.

- **Global Memory Area(글로벌 메모리 영역)**
- **Local Memory Area(Session Memory Area)**

<br>

### Global Memory Area

Mysql 데몬이 뜨는 순간 OS로부터 할당받는 메모리 영역입니다.

모든 쓰레드에서 공유되는 영역입니다.

- **Buffer Pool (InnoDB)**
- **Key Cache (MyISAM)**
- **Query Cache (쿼리 캐시)**
- **Binlog Buffer (바이너리 로그 버퍼)**
- **Log Buffer (로그 버퍼)**
- **Table Cache (테이블 캐시)**

> 대게 1개의 공간으로 할당되지만, 필요에 따라 2개의 공간이 할당될 수 있습니다.

<br>

### Local Memory Area

Client Thread(Foreground Thread(?)) 별로 공간을 할당받는 메모리 영역입니다.

Client Thread 가 사용자의 요청을 처리하기 위해 사용합니다. 

- **Connection 버퍼**
- **Read 버퍼**
- **Result 버퍼**
- **Join 버퍼**
- **Sort 버퍼**
- **Random Read 버퍼**

**'(세션)커넥션이 유지되는 동안 계속해서 할당되어 있는 공간(e.g. Connection 버퍼)' 과 '처리 순간에만 할당되는 공간(e.g. Sort버퍼, Join 버퍼)' 이 있습니다.**

<br><br>

### 참고

[[Real MySQL 정리], 3장 MySQL 아키텍처](https://junghyungil.tistory.com/135)