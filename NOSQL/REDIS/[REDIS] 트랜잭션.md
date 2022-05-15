> *" 여러 자료구조를 사용할 수 있는 Redis 의 특성상 트랜잭션을 잘 이용하면 더 유용하게, 다양한 상황에서 Redis 를 사용할 수 있을 것 입니다. "* <br>
> *출처 : https://sabarada.tistory.com/177*

<br>

Redis 에서는 `MULTI`, `EXEC`, `DISCARD` 와 `WATCH` 명령어를 사용할 수 있다.

|명령어|설명|
|-|-|
|`MULTI`|트랜잭션 시작 <br><br> 트랜잭션 시작 시, 이후 명령어(들)은 바로 실행되지 않고 queue 에 쌓임|
|`EXEC`|queue 에 쌓인 명령어(들)을 일괄적으로 실행<br><br>\* RDBMS 의 commit|
|`DISCARD`|queue 에 쌓인 명령어(들)을 버림<br><br>\* RDBMS 의 rollback|
|`WATCH`|(Optimistic Lock 기반) Lock<br><br>\*WATCH 이후 UNWATCH 되기 전 1번의 `EXEC` 혹은 Transaction 이 아닌 다른 명령어만 허용 <br><br>\* `EXEC` 호출 시 내부적으로 UNWATCH 호출<br>(= 두 번째 `EXEC` 호출 시에는 WATCH 없는 상태)|

<br>

### 참고 사항

- 트랜잭션 과정 중 서버가 다운되었을 경우, 서버 재시작 시 감지되고 오류와 함께 재시작되지 않을 것이다.
- 트랜잭션 과정 중 해당 클라이언트의 커넥션이 끊겼을 경우, 명령어들은 실행되지 않는다. (DISCARD 처리 (?))


<br>

### References

1. [redis.io : Transactions](https://redis.io/docs/manual/transactions/)
2. [[redis] 트랜잭션(Transaction) - 이론편](https://sabarada.tistory.com/177)