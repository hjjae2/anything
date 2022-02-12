## PROPAGATION (전파)

|이름|설명|
|-|-|
|REQUIRED|**Default 설정** <br><br> - 이미 시작된 트랜잭션이 없는 경우 : 트랜잭션 생성 <br>- 이미 시작된 트랜잭션이 있는 경우 : 해당 트랜잭션에 참여|
|REQUIRES_NEW|**항상 새로운(new), 독립된 트랜잭션 시작** <br><br> - 이미 시작된 트랜잭션이 없는 경우 : 트랜잭션 생성 <br> - 이미 시작된 트랜잭션이 있는 경우 : 새로운 트랜잭션 생성, 기존 트랜잭선은 보류(suspend) <br><br> \* 트랜잭션 보류(suspension)의 경우 모든 트랜잭션 매니저(Transaction manager)에서 기본적으로 동작하지 않음에 주의.<br>(TransactionManager to be made available to it)|
|SUPPORTS|- 이미 시작된 트랜잭션이 없는 경우 : 트랜잭션 없이 진행 <br> - 이미 시작된 트랜잭션이 있는 경우 : 해당 트랜잭션에 참여|
|MANDATORY|**기존에 생성된 트랜잭션이 있도록 강제한다.** <br><br> - 이미 시작된 트랜잭션이 없는 경우 : Throw Exception <br> - 이미 시작된 트랜잭션이 있는 경우 : 해당 트랜잭션에 참여 <br>|
|NOT_SUPPORTED|**트랜잭션을 사용하지 않도록 한다.** <br><br> - 이미 시작된 트랜잭션이 없는  경우 : 트랜잭션 없이 진행 <br>- 이미 시작된 트랜잭션이 있는 경우 : 기존 트랜잭션 보류(suspend) <br><br> \* 트랜잭션 보류(suspension)의 경우 모든 트랜잭션 매니저(Transaction manager)에서 기본적으로 동작하지 않음에 주의.<br>(TransactionManager to be made available to it)|
|NEVER|**트랜잭션을 사용하지 않도록 강제한다.** <br><br> - 이미 시작된 트랜잭션이 없는  경우 : 트랜잭션 없이 진행 <br>- 이미 시작된 트랜잭션이 있는 경우 : Throw Exception|
|NEVER|**중첩된 트랜잭션 (REQUIRES_NEW 의 독립적인 트랜잭션과 다름에 주의)** <br><br> - 이미 시작된 트랜잭션이 없는 경우 : 트랜잭션 생성 <br> - 이미 시작된 트랜잭션이 있는 경우 : 중첩 트랜잭션 시작 <br><br> - 기존 트랜잭션(이하 부모 트랜잭션)의 commit/rollback 은 중첩 트랜잭션(이하 자식 트랜잭션)에 영향 <br> - 자식 트랜잭션의 commit/rollback 은 부모 트랜잭션에 영향 X <br><br> **e.g. 자식 트랜잭션에서 exception 에 의해 rollback 발생해도 부모 트랜잭션의 로직은 commit 될 수 있음** <br><br> \* 기본적으로 모든 트랜잭션 메니저(Transaction manager)에서 기본적으로 동작하지 않음에 주의.<br> (일부 JTA provider, JDBC DatasourceTransactionManager 에서 동작)|


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