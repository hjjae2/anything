---
layout: post
title: "DB :: Trigger"
author: "leehyunjae"
tags: ["db"]
---

> 트리거에 대해 찾아본 내용 정리해보기

### 트리거란?

- (데이터베이스에서) 데이터의 입력, 수정, 삭제 등의 이벤트가 발생할 때 자동으로 수행되는 (사용자 정의)프로시저이다.<br>
(즉, 이벤트에 반응하여 실행되는 프로그램)

- 트리거는 TABLE 에 종속되는 개념이 아니라, DATABASE 에 종속되는 개념이다.<br>
(즉, DATABASE에 저장된다.)

- 트리거는 VIEW 가 아닌 실제 TABLE 에 관해서만 작성(정의)할 수 있다.

- 보통 제약조건으로 명시할 수 없는 무결성 제약조건을 지키기 위해(무결성 보장), 혹은 관련 테이블의 데이터를 일치(수정)시켜야 할 때(DB 관리 자동화) 등의 목적으로 사용된다고 한다.

<br>

### 유의 사항

1. Trigger 는 트랜잭션 제어문(COMMIT, ROLLBACK, SAVEPOINT) 을 사용할 수 없다.
2. Trigger 는 걸려있는 대상(트리거링, 테이블이나 ROW를 의미)의 '하나의 실행 부분'으로써 트리거링과 같은 트랜잭션 안에 있다.
3. 2번에 의해 Trigger 가 걸려있는 대상이 commit, rollback 될 때 Trigger 도 commit, rollback 된다.

<br>

### Trigger 의 종류

**1. 문장 트리거**

트리거가 설정된 테이블에 트리거 이벤트가 발생하면, (많은 행에 대해 변경 작업이 발생하더라도) 오직 한번만 트리거를 발생시키는 것이다.

**2. Row 트리거 (행 트리거)**

조건을 만족하는 여러 개의 Row에 대해 트리거를 반복적으로(여러 번) 수행하는 방법이다. Row의 변경 내역은 OLD, NEW(변수) 를 통해 가져올 수 있다.

<br>

### Trigger 표현식

```sql
CREATE [OR REPLACE] TRIGGER '트리거명'
BEFORE|AFTER    -- 트리거가 적용되는 시점, 테이블이 변경되기 전에, 혹은 변경된 후에
(INSERT|UPDATE|DELETE) ON '테이블명'    -- 테이블의 어떤 동작에 대해 적용할건지
[REFERENCING NEW|OLD TABLE AS 테이블명] -- NEW : 새로 추가되거나 변경된 후의 값에 Trigger 적용(INSERT: 삽입할 값, UPDATE: 수정할 값), OLD : 변경 전의 값에 Trigger 적용(UPDATE: 수정 전의 값, DELETE: 삭제할 값)
[FOR EACH ROW]
[WHEN 조건식]   -- Trigger 가 실행되면서 적용될 where 조건
트리거 BODY문   -- Trigger 의 본문 코드 입력, BEIGN ... END 형태이며 적어도 하나의 SQL문이 있어야 한다. 그렇지 않으면 오류를 발생한다고 한다. 변수를 사용할 때는 SET 예약어를 사용한다.? DECLARE 는?
```

```sql
-- [예시]
-- 값이 입력되기 '전'에
-- 점수가 '' 으로 입력되는 것에 대해서 '0'으로 바꿔준다.
-- 점수를 바꿔준 후 INSERT 한다.(?)
CREATE OR REPLACE TRIGGER oracle_trigger
   BEFORE
   INSERT ON oracleStudy
   REFERENCING NEW TABLE AS new_trigger
   FOR EACH ROW
   WHEN new_trigger.점수 = ''
   BEGIN
    SET new_table.점수 = '0';
   END;
```

<br>

### Trigger 관리 명령어

```sql
-- 트리거 활성화/비활성화
ALTER TRIGGER 트리거명 [ENABLE|DISABLE]

-- 테이블에 속한 트리거 활성화/비활성화
-- 트리거는 테이블에 종속,속하는게 아닌데 이것이 가능한건가??
-- 트리거가 가리키고 있는 대상 테이블에서 할 수 있는건가??
ALTER TABLE 테이블명 [ENABLE|DISABLE] ALL TRIGGER

-- 트리거 수정 후 재컴파일
ALTER TRIGGER 트리거명 COMPILE;

-- 트리거 삭제
DROP TRIGGER 트리거명

-- 트리거 조회
SELECT * FROM USER_TRIGGERS;
```