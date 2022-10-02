> 출처 : [트랜잭션 내에 외부 리소스 요청이 담기게 되면 어떤 문제가 발생할까?](https://tecoble.techcourse.co.kr/post/2022-09-20-external-in-transaction/)

## 간단 요약

DB 트랜잭션 시, 꼭 필요한 부분(범위)에만 트랜잭션이 걸릴 수 있도록 한다.

불필요한 부분(예를 들어, DB 트랜잭션과 관련 없는 외부 API 호출 등)은 분리한다.

<br><br>

### 예시

OutGoingService : 외부API호출서비스.class
DBTxService : 꼭필요한DB트랜잭션서비스.class

```java
class DBTxService {
    @Transactional
    void save() { ... }
}
```
```java
class OutGoingService {
    private final DBTxService dbTxService;
    void save() { 
        // 외부 API 호출 로직

        dbTxService.save(); // DB 트랜잭션

        // 외부 API 호출 로직
    }    
}
```