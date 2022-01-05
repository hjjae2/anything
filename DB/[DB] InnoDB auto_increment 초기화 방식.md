### InnoDB 엔진, auto_increment 초기화 방식

(InnoDB 엔진을 사용할 때) Mysql 5.7과 이전버전에서, 서버 재시작 시 auto_increment 의 값은 아래와 같이 초기화된다. 

**이유는 auto_increment 값을 메모리 상에서 관리하기 때문.**

> " In MySQL 5.7 and earlier, **the auto-increment counter is stored in main memory, not on disk.** To initialize an auto-increment counter after a server restart, InnoDB would execute the equivalent of the following statement on the first insert into a table containing an AUTO_INCREMENT column. "

```sql
SELECT MAX(ai_col) FROM table_name FOR UPDATE;
```

<br><br>

### **즉, 아래와 같은 시나리오가 발생할 수 있다.**

1. 데이터에 `1, 2, 3, 4, 5` 라는 값을 넣는다.
   1. 이때 다음 auto_increment 값은 `6` 이다.
2. 데이터 `5`를 지운다.

3. 서버를 재시작한다.
4. 데이터 `6`을 넣는다. 
5. 이때 다음 auto_increment 값 `5`가 된다.


```sql
mysql> select * from test;
+----+-------+
| id | name  |
+----+-------+
|  1 | name1 |
|  2 | name2 |
|  3 | name3 |
|  4 | name4 |
|  5 | name5 |
+----+-------+
5 rows in set (0.01 sec)
```
```sql
mysql> delete from test where id = '5';
Query OK, 1 row affected (0.01 sec)


mysql> select * from test;
+----+-------+
| id | name  |
+----+-------+
|  1 | name1 |
|  2 | name2 |
|  3 | name3 |
|  4 | name4 |
+----+-------+
4 rows in set (0.00 sec)
```
```sql
mysql> insert into test(name) values('name6');
Query OK, 1 row affected (0.00 sec)

mysql> select * from test;
+----+-------+
| id | name  |
+----+-------+
|  1 | name1 |
|  2 | name2 |
|  3 | name3 |
|  4 | name4 |
|  5 | name6 |
+----+-------+
5 rows in set (0.00 sec)
```

<br><br>

### **테이블 설계에 따라 아래와 같이 문제가 발생할 수 있다.**

1. 테이블 A, B가 있다.
2. 테이블 A의 PK는 auto_increment 로 설정되어있다.
3. 테이블 B에서 unique 값으로 A의 PK 값을 갖는다.
4. 테이블 A가 삽입/삭제될 때 테이블 B도 삽입/삭제된다.

**위와 같은 구조에서 테이블 A의 PK 값이 변경되어 원래 예정된 값(`6`)보다 작아졌다면(`5`) 테이블 B에 unique 제약에 걸려 오류가 발생할 수 있다.**

<br><br>

### Mysql 8.0 부터는 동작방식이 변경되어 위와 같은 현상은 발생하지 않는다.

Mysql 8.0 부터는 redo log, data dictionary 라는 곳에 값이 저장되어 영속적으로 관리된다고 한다.

> " In MySQL 8.0, this behavior is changed. The current maximum auto-increment counter value is written to the redo log each time it changes and saved to the data dictionary on each checkpoint. These changes make the current maximum auto-increment counter value persistent across server restarts. "

<br>

### 참고

- [15.6.1.6 AUTO_INCREMENT Handling in InnoDB : InnoDB AUTO_INCREMENT Counter Initialization](https://dev.mysql.com/doc/refman/8.0/en/innodb-auto-increment-handling.html#innodb-auto-increment-initialization)