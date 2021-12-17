> 페이지 로딩 속도 개선한 경험 정리해보기

<br>

### 개요

고객이 메시지를 발송하면 그 기록을 관리툴에서 보여준다. 대략적으로 아래와 같이 생겼다.

<br>

**발송 결과 리스트페이지**

<img src="/images/work/2021-09-02/image_list.png" width="50%" height="50%" alt="발송 결과 리스트">

<br>

**발송 결과 상세페이지**

<img src="/images/work/2021-09-02/image_detail.png" width="50%" height="50%" alt="발송 결과 상세">

<br><br>

### 문제 

**특정 고객이 '발송 결과 상세페이지'에 접근할 때 20~40초 가량의 로딩 시간이 걸린다는 문의가 접수되었다.**

급히 해당 부분에 대한 로직을 살펴보았는데 불필요한 for-loop, method 호출 등 많은 부분들에서 개선이 필요해보였다. 이 중 가장 효과적인 수정이 어떤 것일지 고민했다.

우선 쿼리부터 확인했다. 쿼리는 인덱스 힌트를 통해 인덱스를 탈 수 있도록 되어 있었다. 처음에는 인덱스를 잘 타고 있으니 별 문제가 없겠거니 생각했는데, **실제로 쿼리를 자세히 살펴보니 그렇지 않았다.**

해당 쿼리는 2곳(이하 A, B)에서 호출되고 있었는데, A에서 호출될 때와 B에서 호출될 때의 select, where 절이 달랐다. 위에서 언급한 인덱스 힌트는 A에서 호출할 때에는 적절한 힌트가 되었지만, B에서 호출할 때에는 오히려 적절한 인덱스를 타지 않도록 하고 있었다.

<br>

**단순화하면 다음과 같다.**

```sql
-- A에서 호출할 때 (인덱스 힌트는 고정)

select name
from log use index(name_idx)
where name = ?
```

```sql
-- B에서 호출할 때 (인덱스 힌트는 고정)

select name
from log use index(name_idx)
where id = ?
```

**즉, 인덱싱 처리가 제대로 되고 있지 않았다.**

<br>

### 해결

결론적으로 인덱스 힌트를 수정하여 <u>로딩 시간을 0.1 ~ 3초 정도</u>로 줄일 수 있었다.

```sql
-- A에서 호출할 때
select name
from log use index(name_idx)
where name = ?

-- B에서 호출할 때 (* 실제로는 B에서 호출할 때에는 인덱스 힌트를 주지 않고, optimizer에 의해 선택될 수 있도록 했다.)
select name
from log use index(id_idx)
where id = ?
```

<br>

**아래는 인덱스 힌트 수정 전/후의 실행계획을 살펴본 내용이다.**

> 실제 실행계획 결과는 조금 다르다. 예시를 들기 위해 일부 내용을 수정했다.

```json
{
    "id": 1,
    "select_type": "SIMPLE",
    "table": "r",
    "type": "ref",
    ...
    "key": "IDX_id",
    ...
    "rows": 880,
    "filtered": 1.99,
    "Extra": "Using index condition; Using where"
}
```

```json
{
    "id": 1,
    "select_type": "SIMPLE",
    "table": "r",
    "type": "ref",
    ...
    "key": "IDX_name",
    ...
    "ref": "const",
    "rows": 2730685,
    "filtered": 0,
    "Extra": "Using index condition; Using where;"
}
```