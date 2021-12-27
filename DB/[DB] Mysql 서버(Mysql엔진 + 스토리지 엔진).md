**Mysql 서버는 아래와 같이 구분할 수 있습니다.**
- **Mysql 엔진**
- **스토리지 엔진**

<br><br>

### Mysql 엔진(Mysql Engine)

클라이언트의 요청을 받거나, 쿼리를 파싱하거나 캐싱하는 등의 전반적인 기능을 담당합니다. 

Disk와의 직접적인 접근을 제외한 전반적인 역할을 수행합니다.

- **커넥션 핸들러** : 커넥션 및 쿼리 요청을 처리합니다.
- **SQL 인터페이스** : DML, DDL, Procedure, View 등 SQL 인터페이스를 제공합니다.
- **SQL 파서** : SQL 쿼리 파싱(토큰화)합니다. 이 과정에서 SQL 문법 오류를 탐지할 수 있습니다.
- **SQL 옵티마이저** : 쿼리를 최적화합니다.
- **캐시 & 버퍼** : 성능 향상을 위한 보조 저장소 기능을 담당합니다.

<br><br>

### 스토리지 엔진(Storage Engine)

실제 데이터를 Disk에 저장하거나 Disk로부터 읽어오는 부분을 담당합니다.

스토리지 엔진은 테이블 별로 다르게 설정할 수 있어, 하나의 데이터베이스에서 여러 개를 동시에 사용할 수 있습니다. 

<br>

**InnoDB**

(무결성 보장, 트랜잭션 지원, 오류 복구 기능 제공되기에)비교적 중요한 데이터를 다룰 때, 쓰기 작업이 빈번할 때 권장됩니다.

- (MyISAM에 비해)비교적 많은 기능을 제공하여 무겁습니다. 
  - 상대적으로 작업속도가 느릴 수 있습니다.
  - 시스템 자원을 많이 사용합니다.
- Default로 설정되는 스토리지 엔진입니다.
- **트랜잭션(커밋, 롤백)을 지원합니다.**
- **Row-Level Locking 을 제공합니다.**
- 데이터 복구 기능을 제공합니다.
- 외래키를 지원합니다.
- **인덱스(Index)를 지원합니다.**
- 스토리지 용량 사이즈는 64TB 입니다.

<br>

**MyISAM**

쓰기 작업은 느리고, 읽기(검색)작업은 빠르기 때문에 정적인 데이터를 저장하고 검색하는 데에 적합할 수 있습니다.

- (InnoDB에 비해)기본적인 기능만 제공하여 가볍습니다.
- **트랜잭션을 지원하지 않습니다.**
- **Table Level Locking 을 제공합니다.**
  - **쓰기 작업이 느립니다.**
- **검색(SELECT) 작업이 빠릅니다.**
- 외래키를 지원하지 않습니다.
- **인덱스(Index)를 지원합니다.**
- 스토리지 용량 사이즈는 256TB 입니다.

<br>

**Archive**

데이터가 메모리 상에서 압축되고, 압축된 상태로 디스크에 저장합니다.

- '로그 수집'에 적합합니다.
  - 원시 데이터 관리하는데 효율적일 수 있습니다.
- **Table Level Locking**
- **한번 삽입(Insert)된 데이터는 수정(UPDATE), 삭제(DELETE)할 수 없습니다.**
- **트랜잭션을 지원하지 않습니다.**
- **인덱스(Index)를 지원하지 않습니다.**
- 외래키를 지원하지 않습니다.
- 스토리지 용량 사이즈는 unlimit 입니다.

<br>

**Memory, Federated, ...**

<br><br>

### 핸들러 API(Handler API)

Mysql 엔진이 데이터를 읽기/쓰기 작업을 할 때, 스토리지 엔진에 읽기/쓰기를 요청합니다. 이 요청을 **핸들러 요청**이라고 하고 여기서 사용되는 API를 **핸들러 API**라고 합니다.

즉, 핸들러 API 를 통해 스토리지 엔진에 작업을 요청할 수 있습니다.

```sql
show global status like 'Handler%';
```
```text
+--------------------------+-----------+
|Variable_name             |Value      |
+--------------------------+-----------+
|Handler_commit            |145753485  |
|Handler_delete            |420139     |
|Handler_discover          |0          |
|Handler_external_lock     |378322641  |
|Handler_mrr_init          |0          |
|Handler_prepare           |21529250   |
|Handler_read_first        |49551083   |
|Handler_read_key          |422580944  |
|Handler_read_last         |9891       |
|Handler_read_next         |351618495  |
|Handler_read_prev         |33289560   |
|Handler_read_rnd          |42499941   |
|Handler_read_rnd_next     |20689540902|
|Handler_rollback          |2647       |
|Handler_savepoint         |1          |
|Handler_savepoint_rollback|0          |
|Handler_update            |711264     |
|Handler_write             |10041446135|
+--------------------------+-----------+
```
- Variable_name : 핸들러 API 이름
- Value : 해당 API 를 통해 작업된 레코드의 수

<br><br>

### 참고

- [[MySQL] 주요 스토리지 엔진(Storage Engine) 간단 비교](http://asuraiv.blogspot.com/2017/07/mysql-storage-engine.html)
- [[MySQL] Engine 별 기능 알아보기](https://mozi.tistory.com/91)