Kafka Broker : 카프카가 설치된 서버의 단위를 의미
- 보통 브로커 3대 유지하기를 권장한다고 함

<br>

partition -> replication 의 수 만큼 복제 (단, 브로커 개수에 제한된다.)
- Leader partition (DB의 master 개념)
  - producer의 데이터를 전달받는 주체
- Follower parition (DB의 slave 개념)
  - Leader partition(브로커) 죽었을 때 -> follower partition 이 Leader 가 되어 가용성 보장
- ISR = Leader + Follower partition

> e.g. replciation 3 이라면, Leader partition 1, Follower partition 2 개 존재한다.

<br>

**Producer 의 ack 옵션**

- ack 0 : 데이터 전달 후 응답 받지 않는다. 데이터가 잘 전송되었는지 알 수 없다.
- ack 1 : 데이터 전달 후 Leader partition 에 대해서만 응답 값을 받는다. Leader partition 에 대해서는 데이터가 잘 전송되었는지 알 수 있다. 단, Leader partition 에 저장된 것이 Follower 에도 잘 저장된 것을 의미하지는 않는다. 즉, 데이터 유실 가능성은 아직도 있다.
- ack all : Leader, Follower 파티션 모두에 응답 값을 받는다. 데이터 유실 가능성은 없다. 다만 속도가 현저히 늦어진다.

> 데이터 유입량, 유지시간 등을 고려해 replication 수를 지정한다.
> 
> 3개 이상의 브로커를 사용할 때 replication 3 설정을 권장한다.