### SELECT 쿼리 실행 순서

```text
드라이빙 테이블 (where 적용 / 조인 실행)
드리븐 테이블1 
드리븐 테이블2
-> GROUP BY 
-> DISTINCT
-> HAVING
-> ORDER BY
-> LIMIT
```

```text
// 주로 group by 없이 order by만 적용된 쿼리에서 사용될 수 있는 순서이다.

드라이빙 테이블
-> ORDER BY
-> 드리븐 테이블1, 드리븐 테이블2 (were 적용 / 조인 실행)
-> LIMIT 적용
```

<br>

### 인덱스 컬럼의 값을 변환/조작하지 않는다.

**인덱스를 사용하지 않는 경우**

```sql
# user_id 가 index라고 가정한다.
explain select * from poster where user_id * 1 = 12;

+--+-----------+------+----------+----+-------------+----+-------+----+----+--------+-----------+
|id|select_type|table |partitions|type|possible_keys|key |key_len|ref |rows|filtered|Extra      |
+--+-----------+------+----------+----+-------------+----+-------+----+----+--------+-----------+
|1 |SIMPLE     |poster|NULL      |ALL |NULL         |NULL|NULL   |NULL|17  |100     |Using where|
+--+-----------+------+----------+----+-------------+----+-------+----+----+--------+-----------+
```

<br>

**인덱스를 사용하는 경우**
```sql
# user_id 가 index라고 가정한다.
explain select * from poster where user_id = 12;

+--+-----------+------+----------+----+-------------+-------+-------+-----+----+--------+-----+
|id|select_type|table |partitions|type|possible_keys|key    |key_len|ref  |rows|filtered|Extra|
+--+-----------+------+----------+----+-------------+-------+-------+-----+----+--------+-----+
|1 |SIMPLE     |poster|NULL      |ref |user_id      |user_id|9      |const|2   |100     |NULL |
+--+-----------+------+----------+----+-------------+-------+-------+-----+----+--------+-----+
```

<br>

### OR 연산자는 주의한다.

or 연산자는 row 수가 더 늘어나기에 테이플 풀스캔이 발생할 수 있다. 주의하여 사용한다.

<br>

### GROUP BY

- (복합 인덱스의 경우) where 절과 달리 group by 절에서는 컬럼의 순서가 같아야한다. 
  - 뒤쪽 컬럼의 경우, 없어도 인덱스를 탈 수 있다.
  - **group by 절에, 인덱스에 없는 컬럼이 하나라도 있다면 인덱스를 탈 수 없다.**

```sql
-- 인덱스가 (COL1, COL2, COL3, COL4)라고 가정했을 때, 아래의 경우 인덱스를 탈 수 있다.
... GROUP BY COL1
... GROUP BY COL1, COL2
... GROUP BY COL1, COL2, COL3
... GROUP BY COL1, COL2, COL3, COL4
```

<br>

- (앞쪽 컬럼이) where 절에 동등 비교 조건으로 컬럼이 들어가면, group by 에서 빠져도 될 수도 있다.

```sql
explain select started_date from poster group by user_id, started_date;

+--+-----------+------+----------+-----+--------------------+--------------------+-------+----+----+--------+------------------------+
|id|select_type|table |partitions|type |possible_keys       |key                 |key_len|ref |rows|filtered|Extra                   |
+--+-----------+------+----------+-----+--------------------+--------------------+-------+----+----+--------+------------------------+
|1 |SIMPLE     |poster|NULL      |range|user_id_started_date|user_id_started_date|13     |NULL|5   |100     |Using index for group-by|
+--+-----------+------+----------+-----+--------------------+--------------------+-------+----+----+--------+------------------------+
```

```sql
-- 인덱스 (user_id, started_date) 가정한다.
-- where 절에 앞쪽 컬럼, group by 에 뒤쪽 컬럼인 경우
explain select started_date from poster where user_id = 12 group by started_date;

+--+-----------+------+----------+----+-----------------------------------------+--------------------+-------+-----+----+--------+------------------------+
|id|select_type|table |partitions|type|possible_keys                            |key                 |key_len|ref  |rows|filtered|Extra                   |
+--+-----------+------+----------+----+-----------------------------------------+--------------------+-------+-----+----+--------+------------------------+
|1 |SIMPLE     |poster|NULL      |ref |user_id,user_id_started_date,started_date|user_id_started_date|9      |const|2   |100     |Using where; Using index|
+--+-----------+------+----------+----+-----------------------------------------+--------------------+-------+-----+----+--------+------------------------+

-- 인덱스 (started_date, user_id) 사용한 경우
+--+-----------+------+----------+-----+--------------------------------------------------------------+--------------------+-------+----+----+--------+-------------------------------------+
|id|select_type|table |partitions|type |possible_keys                                                 |key                 |key_len|ref |rows|filtered|Extra                                |
+--+-----------+------+----------+-----+--------------------------------------------------------------+--------------------+-------+----+----+--------+-------------------------------------+
|1 |SIMPLE     |poster|NULL      |range|user_id,user_id_started_date,started_date,started_date_user_id|started_date_user_id|13     |NULL|5   |100     |Using where; Using index for group-by|
+--+-----------+------+----------+-----+--------------------------------------------------------------+--------------------+-------+----+----+--------+-------------------------------------+

```

```sql
-- 인덱스 (user_id, started_date) 가정한다.
-- where 절에 뒤쪽 컬럼, group by 에 앞쪽 컬럼인 경우
explain select user_id from poster where started_date = '2020-12-24' group by user_id;

+--+-----------+------+----------+-----+-----------------------------------------+--------------------+-------+----+----+--------+-------------------------------------+
|id|select_type|table |partitions|type |possible_keys                            |key                 |key_len|ref |rows|filtered|Extra                                |
+--+-----------+------+----------+-----+-----------------------------------------+--------------------+-------+----+----+--------+-------------------------------------+
|1 |SIMPLE     |poster|NULL      |range|user_id,user_id_started_date,started_date|user_id_started_date|13     |NULL|3   |100     |Using where; Using index for group-by|
+--+-----------+------+----------+-----+-----------------------------------------+--------------------+-------+----+----+--------+-------------------------------------+
```

<br>

### ORDER BY

group by와 유사하다. 다만 asc, desc 를 통일시켜야 한다.

```sql
explain select poster_id from poster where user_id = 12 order by started_date;

+--+-----------+------+----------+----+----------------------------+--------------------+-------+-----+----+--------+------------------------+
|id|select_type|table |partitions|type|possible_keys               |key                 |key_len|ref  |rows|filtered|Extra                   |
+--+-----------+------+----------+----+----------------------------+--------------------+-------+-----+----+--------+------------------------+
|1 |SIMPLE     |poster|NULL      |ref |user_id,user_id_started_date|user_id_started_date|9      |const|18  |100     |Using where; Using index|
+--+-----------+------+----------+----+----------------------------+--------------------+-------+-----+----+--------+------------------------+
```

```sql
explain select poster_id from poster order by user_id, started_date;

+--+-----------+------+----------+-----+-------------+--------------------+-------+----+----+--------+-----------+
|id|select_type|table |partitions|type |possible_keys|key                 |key_len|ref |rows|filtered|Extra      |
+--+-----------+------+----------+-----+-------------+--------------------+-------+----+----+--------+-----------+
|1 |SIMPLE     |poster|NULL      |index|NULL         |user_id_started_date|13     |NULL|18  |100     |Using index|
+--+-----------+------+----------+-----+-------------+--------------------+-------+----+----+--------+-----------+
```

```sql
explain select poster_id from poster where started_date = '2020-12-24' order by user_id;

+--+-----------+------+----------+----+-------------+------------+-------+-----+----+--------+-------------------------------------+
|id|select_type|table |partitions|type|possible_keys|key         |key_len|ref  |rows|filtered|Extra                                |
+--+-----------+------+----------+----+-------------+------------+-------+-----+----+--------+-------------------------------------+
|1 |SIMPLE     |poster|NULL      |ref |started_date |started_date|4      |const|12  |100     |Using index condition; Using filesort|
+--+-----------+------+----------+----+-------------+------------+-------+-----+----+--------+-------------------------------------+

-- 비교를 위해, (start_date, user_id) 인덱스인 경우는 아래와 같다.
+--+-----------+------+----------+----+---------------------------------+--------------------+-------+-----+----+--------+------------------------+
|id|select_type|table |partitions|type|possible_keys                    |key                 |key_len|ref  |rows|filtered|Extra                   |
+--+-----------+------+----------+----+---------------------------------+--------------------+-------+-----+----+--------+------------------------+
|1 |SIMPLE     |poster|NULL      |ref |started_date,started_date_user_id|started_date_user_id|4      |const|12  |100     |Using where; Using index|
+--+-----------+------+----------+----+---------------------------------+--------------------+-------+-----+----+--------+------------------------+

```

<br>

### WHERE 조건 + ORDER BY (or GROUP BY)

**1. where절 + order by절이 동시에 같은 인덱스 사용**

where 조건에 들어가는 컬럼, order by 절에 들어가는 컬럼이 모두 하나의 인덱스에 포함되어 있을 때이다.

물론 order by가 있기 때문에 order by의 순서도 지켜져야할 것이다.

**2. where절만 인덱스 사용**

were 절만 인덱스를 통해 데이터를 조회하고, order by 절은 인덱스를 사용하지 않는다. 즉, order by 쪽은 filesort 를 통해 정렬한다.

**where 절 이후 레코드 수가 없을 경우 효율적일 것이다.**

**3. order by절만 인덱스 사용**

order by 절은 인덱스를 사용한다. 

즉, 인덱스 통해 이미 정렬되어 있는 데이터를 순서대로 읽으면서 where 절의 조건에 일치하는지 확인하는 것이다. 일차히지 않으면 버리는 형태이다.

<img src="https://t1.daumcdn.net/cfile/tistory/2110023A587EED9E1D" alt="출처:https://weicomes.tistory.com/191?category=669169">

> " 위 그림은 WHERE 과 ORDER BY 절이 결합된 두 가지 패턴의 쿼리를 표현한 것이다. 그림 오른쪽과 같이 ORDER BY  절에 해당 칼럼이 사용되고 있다면 WHERE 절에 동등 비교 이외의 연산자로 비교돼도 WHERE 조건과 ORDER BY 조건이 모두 인덱스를 이용할 수 있다. 위 그림의 왼쪽 패턴 쿼리 예제는 다음과 같다. "
> 
> 출처 : https://weicomes.tistory.com/191?category=669169

<br>

### ORDER BY + GROUPO BY

하나의 인덱스를 사용하려면, 두 절(order by, group by)모두 인덱스 컬럼과 동일하게, 순서대로 작성되어야 한다. 

**둘 중 하나라도 인덱스를 이용하지 못하면 둘 다 인덱스를 타지 못한다.**

<br>

### WHERE + ORDER BY + GROUP BY (:star::star:)

<img src="https://t1.daumcdn.net/cfile/tistory/2652B433587EEDA02B" alt="출처:https://weicomes.tistory.com/191?category=669169">

> 출처 : https://weicomes.tistory.com/191?category=669169

<br><br>

### 참고

1. [[mysql]인덱스 정리 및 팁](https://jojoldu.tistory.com/243)
1. [28. MySQL 실행 계획 : 실행 계획 분석(6)](https://velog.io/@jsj3282/28.-MySQL-실행-계획-실행-계획-분석6)