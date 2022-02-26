> 정리하면 아래와 같을 수 있을 것 같음
>
> - 낙관적 락 : CAS
> 
> - 비관적 락 : Lock

<br>

## Optimistic Lock (낙관적 락)

**트랜잭션 충돌이 발생하지 않는다고 가정하는 것**

DB 제공하는 Lock 을 사용하지 않고, Application Level(e.g. JPA)에서 버전 관리 직접함

**> 버전 관리 : version, timestamp, hashcode 등**

> CAS 개념으로 볼 수 있을 것 같다.

Lock 을 사용하지 않기 때문에, 트랜잭션 충돌이 많지 않은 상황에 사용하면 성능 향상

> Lock vs CAS 의 비교로도 볼 수 있을 듯

```java
...
@Entity
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    ...

    @Version
    private Long version;
}
```

<br>

## Pessimistic Lock (비관적 락)

**트랜잭션 충돌이 발생한다고 가정하는 것**

> 트랜잭션이 충돌이 발생한다고 가정하고, 우선적으로 Lock 을 거는 것

트랜잭션 충돌이 발생하는 것을 가정하기에, Lock(공유 잠금, 배타 잠금)을 건다.

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    @Lock(LockModeType.PESSIMISTIC_WRITE)
    @Query("~")
    Member findByName(String name);
}
```

<br>
<br>

## 참고 : 공유 잠금, 배타 잠금 예시

```sql
-- 공유 잠금
SELECT * FROM mytable WHERE id = 1 LOCK IN SHARE MODE;
 
--  배타 잠금
SELECT * FROM mytable WHERE id = 1 FOR UPDATE;
```