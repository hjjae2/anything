## ReadFrom

Defines from which Redis nodes data is read.

![](../images/[ETC]%20Lettuce%20ReadFrom_07.png)

<br>

## read 커맨드, write 커맨드 식별 방법

> `io.lettuce.core.cluster.ReadOnlyCommands` 클래스 참고

redis 커맨드를 기준으로 식별한다.

![](../images/[ETC]%20Lettuce%20ReadFrom_47.png)

<br>

### lua script

스크립트는 eval, evalsha 커맨드로 실행 -> `read` 커맨드로 판단한다.

<br>

## REPLICA_PREFERRED vs MASTER_PREFERRED

1. master(write)커넥션, replica(reader)커넥션을 리스트에 저장한다.
2. reader 커넥션을 가져올 때 1번의 리스트 중 0번째 인덱스에서 가져온다.

|분류|컬렉션 내 커넥션 순서|
|-|-|
|`REPLICA_PREFERRED`|0번째 인덱스 : reader <br> 1번째 인덱스 : writer|
|`MASTER_PREFERRED`|0번째 인덱스 : writer <br> 1번째 인덱스 : reader|

<br>

**MASTER_PREFERRED**

![](../images/[ETC]%20Lettuce%20ReadFrom_26.png)

**REPLICA_PREFERRED**

![](../images/[ETC]%20Lettuce%20ReadFrom_34.png)

