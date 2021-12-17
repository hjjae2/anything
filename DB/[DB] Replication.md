---
layout: post
title: "DB :: Replication"
author: "leehyunjae"
tags: ["db"]
---

> Replication 에 대해 이해해보기

### Replication

> 회사에서 대용량의, 중요한 테이블에 대한 스키마 변경 작업을 하게 되었다. 작업을 진행하기 전 Replication 에 대한 개념이 부족하여 replication 에 대해 공부해보고 정리해보고자 한다.

Replication 은 말그대로 '복제'이다. DB에 데이터를 insert, update, delete 하면 그 내용을 그대로 (여분의, 추가의)DB에 저장해놓는 것이다.

이렇게 복제된 DB는 백업용으로 사용할 수도 있고, select query를 처리해줄 수도 있다. 예를 들어, select query를 slave DB에 위임하면 메인 DB의 부하를 감소시킬 수 있을 것이다.
보통의 경우 (성능을 고려하여)master DB에 insert, update, delete 동작을 실행하고, slace DB에 select 쿼리를 질의한다고 한다. 

<br>

### 알아야 할 것

**binary log (binlog)**

slave DB가 master DB의 내용을 복제할 때 어떻게 복제할까? binlog 파일을 보고 복제한다고 한다. binlog는 mysql에서 실행된 모든 내용이 기록되는 파일이라고 한다. mysql은 이 binlog 와 binlog의 position(pos) 값을 가지고 있는데, 이 값들을 토대로 slave DB가 복제할 수 있는 것이다.

아래와 같이 명령어를 통해 binlog(현재 파일명)와 binlog의 position 값을 알 수 있다. 이 값을 slave 도 동일하게 바라보고 있는 것이다.

```sql
mysql> show master status;
+---------------+----------+--------------+------------------+-------------------+
| File          | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+---------------+----------+--------------+------------------+-------------------+
| binlog.000004 |      156 |              |                  |                   |
+---------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```

<br>

**binlog의 내용을 간단히 살펴보면 다음과 같다.**

먼저 mysql 의 binlog 파일이 있는 경로로 이동한다. (환경마다 다르기에, 인터넷에서 경로를 찾아 확인하면 된다.)

```sh
> cd /opt/homebrew/var/mysql

> ls
...
binlog.000003
binlog.000004
...
```

그럼 아래와 같이 binlog 의 내용을 볼 수 있다. cat 과 같은 명령어를 사용해서는 읽을 수 없고 mysqlbinlog 명령어를 통해 읽을 수 있다.

```sh
> mysqlbinlog binlog.000004

# The proper term is pseudo_replica_mode, but we use this compatibility alias
# to make the statement usable on server versions 8.0.24 and older.
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=1*/;
/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
DELIMITER /*!*/;
# at 4

...

ROLLBACK/*!*/;

...

# [empty]
SET @@SESSION.GTID_NEXT= 'AUTOMATIC' /* added by mysqlbinlog */ /*!*/;
DELIMITER ;
# End of log file
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;
```

<br>

그리고 이때의 binlog, position 값은 다음과 같다.

```sh
mysql> show master status;
+---------------+----------+--------------+------------------+-------------------+
| File          | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+---------------+----------+--------------+------------------+-------------------+
| binlog.000004 |      156 |              |                  |                   |
+---------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```

<br>

그리고 테스트로 생성한 DB에 insert 쿼리를 날려보면 다음과 같이 값이 추가되어 있음을 확인할 수 있다.

```sh
> mysqlbinlog binlog.000004

# The proper term is pseudo_replica_mode, but we use this compatibility alias
# to make the statement usable on server versions 8.0.24 and older.
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=1*/;
/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
DELIMITER /*!*/;
# at 4

...

ROLLBACK/*!*/;

...

# [empty]
SET @@SESSION.GTID_NEXT= 'AUTOMATIC' /* added by mysqlbinlog */ /*!*/;
DELIMITER ;
# End of log file
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;

################################ 여기부터는 추가된 내용이다.
# at 156

...

SET TIMESTAMP=1636549521/*!*/;
SET @@session.pseudo_thread_id=11/*!*/;
SET @@session.foreign_key_checks=1, @@session.sql_auto_is_null=0, @@session.unique_checks=1, @@session.autocommit=1/*!*/;

...

SET @@session.auto_increment_increment=1, @@session.auto_increment_offset=1/*!*/;
/*!\C utf8mb4 *//*!*/;
SET @@session.character_set_client=255,@@session.collation_connection=255,@@session.collation_server=255/*!*/;
SET @@session.lc_time_names=0/*!*/;
SET @@session.collation_database=DEFAULT/*!*/;

...

# End of log file
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;
```

<br>

이때의 binlog, position 값은 다음과 같고 position 의 값이 올라간 것을 알 수 있다.

```sh
mysql> show master status;
+---------------+----------+--------------+------------------+-------------------+
| File          | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+---------------+----------+--------------+------------------+-------------------+
| binlog.000004 |      488 |              |                  |                   |
+---------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```

<br>

### 참고

1. [MySQL Master-Slave 복제소개 및 사용방법](https://myinfrabox.tistory.com/22)