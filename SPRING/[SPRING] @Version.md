## `@Version`

> '낙관적 락' 구현 시 사용될 수 있음

```java
package javax.persistence;

...

/**
 * Specifies the version field or property of an entity class that serves as its optimistic lock value.
 *
 * The version is used to ensure integrity when performing the merge operation and for optimistic concurrency control.
 *
 * Only a single Version property or field should be used per class; applications that use more than one Version property or field will not be portable.
 *
 * ...
 *
 * The following types are supported for version properties: int, Integer, short, Short, long, Long, java.sql.Timestamp.
 */
@Target({METHOD, FIELD})
@Retention(RUNTIME)
public @interface Version {}
```

<br>

**1. 자바 표준 애노테이션**

> 패키지 : javax.persistence;

<br>


**2. 클래스 당 오직 하나의 `@Version` 필드, 컬럼**

<br>

**3. 지원 타입**

- Integer (int)
- Short (short)
- Long (long)
- Timestamp

<br>

**4. 수정 시 (조회했을 때의)Version 값이 다르다면, 예외 발생**

> CAS

```sql
update mytable set ~~~ where ~~~ and version = ?
```

```sql
select
    customerve0_.id as id1_1_0_,
    customerve0_.name as name2_1_0_,
    customerve0_.version as version3_1_0_ 
from
    customer_version customerve0_ 
where
    customerve0_.id=?
-- 업데이트 전 : CustomerVersion(id=1, name=c1, version=0)


update
    customer_version 
set
    name=?,
    version=? 
where
    id=? 
    and version=?
-- 업데이트 후 : CustomerVersion(id=1, name=name1, version=1)
```
