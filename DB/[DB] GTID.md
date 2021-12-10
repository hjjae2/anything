---
layout: post
title: "DB :: GTID (with CTAS)"
author: "leehyunjae"
tags: ["db"]
---

> GTID 에 대해 이해해보기

### GTID(Global Transaction IDentifier) (작성중)

형태 : `고유한 식별자(ID):TRANSACTION ID`

- Master,Slave 복제 기준이 되었던 binlog, pos 대신 GTID 를 사용할 수 있다.
  - Master 의 binglog file, pos 를 따라가지 않아도 된다.
- GTID 정보만으로 Master, Slave 간의 일관성을 쉽게 확인할 수 있다.
- GTID 트랜잭션은 mysql.gtid_executed 로 관리되기에, 중복 수행되지 않는다.

1. 트랜잭션이 commit 되면 GTID 를 할당 받고 binlog 에 기록한다.
2. 할당된 GTID 는 gtid_executed system variable, mysql.gtid_executed 에 저장된다.

> (Mysql 5.7 기준)GTID 에서는 `CTAS`, `CREATE TEMPORARY TABLE` 실행 불가와 같은 몇 가지 제약사항이 있다. ([참고](https://dev.mysql.com/doc/refman/5.7/en/replication-gtids-restrictions.html))