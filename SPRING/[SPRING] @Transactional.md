## PROPAGATION (전파)

|이름|설명|
|-|-|
|REQUIRED|**Default 설정** <br><br> - 이미 시작된 트랜잭션이 없는 경우 : 트랜잭션 생성 <br>- 이미 시작된 트랜잭션이 있는 경우 : 해당 트랜잭션에 참여|
|REQUIRES_NEW|**항상 새로운(new), 독립된 트랜잭션 시작** <br><br> - 이미 시작된 트랜잭션이 없는 경우 : 트랜잭션 생성 <br> - 이미 시작된 트랜잭션이 있는 경우 : 새로운 트랜잭션 생성, 기존 트랜잭선은 보류(suspend) <br><br> \* 트랜잭션 보류(suspension)의 경우 모든 트랜잭션 매니저(Transaction manager)에서 기본적으로 동작하지 않음에 주의.<br>(TransactionManager to be made available to it)|
|SUPPORTS|- 이미 시작된 트랜잭션이 없는 경우 : 트랜잭션 없이 진행 <br> - 이미 시작된 트랜잭션이 있는 경우 : 해당 트랜잭션에 참여|
|MANDATORY|**기존에 생성된 트랜잭션이 있도록 강제한다.** <br><br> - 이미 시작된 트랜잭션이 없는 경우 : Throw Exception <br> - 이미 시작된 트랜잭션이 있는 경우 : 해당 트랜잭션에 참여 <br>|
|NOT_SUPPORTED|**트랜잭션을 사용하지 않도록 한다.** <br><br> - 이미 시작된 트랜잭션이 없는  경우 : 트랜잭션 없이 진행 <br>- 이미 시작된 트랜잭션이 있는 경우 : 기존 트랜잭션 보류(suspend) <br><br> \* 트랜잭션 보류(suspension)의 경우 모든 트랜잭션 매니저(Transaction manager)에서 기본적으로 동작하지 않음에 주의.<br>(TransactionManager to be made available to it)|
|NEVER|**트랜잭션을 사용하지 않도록 강제한다.** <br><br> - 이미 시작된 트랜잭션이 없는  경우 : 트랜잭션 없이 진행 <br>- 이미 시작된 트랜잭션이 있는 경우 : Throw Exception|
|NESTED|**중첩된 트랜잭션 (REQUIRES_NEW 의 독립적인 트랜잭션과 다름에 주의)** <br><br> - 이미 시작된 트랜잭션이 없는 경우 : 트랜잭션 생성 <br> - 이미 시작된 트랜잭션이 있는 경우 : 중첩 트랜잭션 시작 <br><br> - 기존 트랜잭션(이하 부모 트랜잭션)의 commit/rollback 은 중첩 트랜잭션(이하 자식 트랜잭션)에 영향 <br> - 자식 트랜잭션의 commit/rollback 은 부모 트랜잭션에 영향 X <br><br> **e.g. 자식 트랜잭션에서 exception 에 의해 rollback 발생해도 부모 트랜잭션의 로직은 commit 될 수 있음** <br><br> \* 기본적으로 모든 트랜잭션 메니저(Transaction manager)에서 기본적으로 동작하지 않음에 주의.<br> (일부 JTA provider, JDBC DatasourceTransactionManager 에서 동작)|


<br><br>

## ISOLATION (격리)

> 각 격리 수준의 설명은 DB 격리 수준의 내용과 동일하기에 생략

|이름|설명|
|-|-|
|DEFAULT|Datasource 설정 따름 <br> Use the default isolation level of the underlying datastore|
|READ_UNCOMMITED||
|READ_COMMITED||
|REPEATABLE_READ||
|SERIALIZABLE||

<br><br>

## timeout 

The timeout for this transaction (in seconds).


기본 값 : 시스템의 트랜잭션 타임아웃 값

> Defaults to the default timeout of the underlying transaction system.

**'새롭게 시작된 트랜잭션'에만 적용할 수 있기 때문에, `REQUIRED`, `REQUIRES_NEW`와 함께 사용하도록 설계됨**

> Exclusively designed for use with Propagation.REQUIRED or Propagation.REQUIRES_NEW since it only applies to newly started transactions.

<br><br>

## readOnly

\-

<br><br>

## rollbackFor, rollbackForClassName / noRollbackFor, noRollbackForClassName

rollback, noRollback 대상 지정


<br>

**[참고] Transaction 롤백 대상에 관하여,**

> " By default, a transaction will be rolling back on RuntimeException and Error but not on checked exceptions (business exceptions). See org.springframework.transaction.interceptor.DefaultTransactionAttribute.rollbackOn(Throwable) for a detailed explanation. "

DefaultTransactionAttribute 내용을 살펴보면, 다음과 같다.

RuntimeException, Error = unexpected outcome
CheckedException = expected outcome

```java

public class DefaultTransactionAttribute extends DefaultTransactionDefinition implements TransactionAttribute {
    ...

    /**
     * The default behavior is as with EJB: rollback on unchecked exception (RuntimeException), assuming an unexpected outcome outside of any business rules. 
     * Additionally, we also attempt to rollback on Error which is clearly an unexpected outcome as well.
     * By contrast, a checked exception is considered a business exception and therefore a regular expected outcome of the transactional business method, i.e. a kind of alternative return value which still allows for regular completion of resource operations.
     */

    @Override
	public boolean rollbackOn(Throwable ex) {
		return (ex instanceof RuntimeException || ex instanceof Error);
	}

    ...
}
```

<br><br>

## 그 외

https://techblog.woowahan.com/2606/ <br>
> "  여기서 주목할 만한 사실은, 전파속성(*propagation*) 때문에 실제 트랜잭션이 재사용되더라도 트랜잭션 메서드의 반환시점마다 트랜잭션의 완료처리(completion)를 한다는 것입니다. 물론 커밋이나 롤백같은 최종완료처리는 최초 트랜잭션이 반환될 때 일어나겠지만요. "

요약 : 트랜잭션 완료처리 시점(메서드 반환)에 (unchecked)Exception이 터지면 rollback 마킹
