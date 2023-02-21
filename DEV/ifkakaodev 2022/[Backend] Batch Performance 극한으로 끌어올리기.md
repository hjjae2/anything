> 배치 코드의 경우 (비교적)운영 중 모니터링, 성능에 관심을 기울이지 않게 되는 것 같다. 

<br>

## READ

> 성능 개선의 첫 걸음, Reader 개선

대부분의 배치에서 writer 보다 reader 가 차지하는 비중이 크다.

![](../../images/[Backend]%20Batch%20Performance%20극한으로%20끌어올리기_27.png)

예를 들어, 위 그림과 같이 10억 건 중 100만 건을 추출해야 할 때 select 쿼리의 수정만으로도 큰 효과를 볼 수 있을 것이다.

<br>

### Chunk Processing

> 대용량에서는 chunk processing 이 필수이다. (절대적으로 사용될 것이다.)

![](../../images/[Backend]%20Batch%20Performance%20극한으로%20끌어올리기_15.png)

![](../../images/[Backend]%20Batch%20Performance%20극한으로%20끌어올리기_56.png)

**문제점 : Pagination**

Pagination 을 주로 사용할텐데 성능 관점에서 좋지 않다.

offset이 커질 수록 비용이 커진다.

![](../../images/[Backend]%20Batch%20Performance%20극한으로%20끌어올리기_43.png)


<br>

**개선 방법 1 : ZeroOffsetItemReader**

ZeroOffsetItemReader 를 (먄들어)사용한다. 

offset 은 항상 0으로 유지하고, where 절을 변경한다.

![](../../images/[Backend]%20Batch%20Performance%20극한으로%20끌어올리기_11.png)

![](../../images/[Backend]%20Batch%20Performance%20극한으로%20끌어올리기_57.png)

<br>

**개선 방법 2 : Cursor**

![](../../images/[Backend]%20Batch%20Performance%20극한으로%20끌어올리기_49.png)

> Jpa 말고 Jdbc 커서를 추천한다. 

다만 쿼리를 문자열로 작성해야 한다는 문제점이 있다. 

exposed 를 사용하고 있다.

![](../../images/[Backend]%20Batch%20Performance%20극한으로%20끌어올리기_30.png)

<br>

### ItemReader 성능 비교

![](../../images/[Backend]%20Batch%20Performance%20극한으로%20끌어올리기_34.png)

핵심 : JpaPagingItemReader 만 개수에 비례하여 시간이 증가되지 않는다.

![](../../images/[Backend]%20Batch%20Performance%20극한으로%20끌어올리기_25.png)

성능 뿐만 아니라 안정성도 검증되었다고 볼 수 있다.

<br>

### 정리

![](../../images/[Backend]%20Batch%20Performance%20극한으로%20끌어올리기_32.png)


![](../../images/[Backend]%20Batch%20Performance%20극한으로%20끌어올리기_42.png)

<br><br>

## Aggregation

sum, group by 등의 쿼리를 사용하는 경우가 빈번할 것이다.

이 쿼리들은 비용이 클 수 있다. (꼭 실행 계획을 확인해보자.)

![](../../images/[Backend]%20Batch%20Performance%20극한으로%20끌어올리기_28.png)

![](../../images/[Backend]%20Batch%20Performance%20극한으로%20끌어올리기_44.png)

> 쿼리 자체가 비효율적이라면 앞서 개선한 ItemReader 도 무용지물이 될 것이다.

<br>

### Group By 포기

이 팀에서는 group by를 포기했다. 

쿼리는 단순하게 작성하고 애플리케이션 단에서 aggregation 한다.

**문제점 : 애플리케이션 단에서 집계하기 위해 데이터를 가져와야하는데 OOM 유발 가능성이 커진다.**

<br>

### 새로운 Architecture (for 애플리케이션 집계)

> group by 를 포기하고 애플리케이션 레이어에서의 집계를 위해 아키텍쳐를 설계했다.

![](../../images/[Backend]%20Batch%20Performance%20극한으로%20끌어올리기_29.png)

![](../../images/[Backend]%20Batch%20Performance%20극한으로%20끌어올리기_12.png)

<br>

**문제점 : Network I/O**

> **개선 : Redis Pipeline**

![](../../images/[Backend]%20Batch%20Performance%20극한으로%20끌어올리기_35.png)

<br><br>

## Write

write 성능 개선을 위해 두 가지 핵심 포인트가 있다.

|핵심 포인트|설명|
|-|-|
|Batch Insert|- 일괄 쿼리 요청|
|명시적 쿼리|- 필요한 컬럼만 Update <br> - 영속성 컨텍스트 사용 X|

**결론**

(아쉽지만) JPA(영속성 관리, Dirty Checking 등)를 버려야한다.

<br>

### Batch 에서 JPA Write 에 대한 고찰

> Batch 환경에서 JPA 가 잘 맞는지

**1. 불필요한 Dirty Checking & 영속성 관리**

실제로 reader 에서도 영속성 관리되지 않게 사용하고 있다. DTO로 반환받는다.

(불필요한 check 로직으로 인해) 큰 성능 저하를 유발시킬 수 있다.

**2. Update 할 때 불필요한 컬럼도 업데이트 (쿼리에 포함)**

쿼리 statement 가 커진다. jpa dynamic update 는 오히려 성능이 저하된다. 

소폭의 성능 저하를 유발시킬 수 있다.

**3. Jpa Batch Insert 지원이 어렵다.**

Identity 의 경우, batch insert 를 지원하지 않는다. (물론 다른 채번 방법을 사용하면 되긴 한다.)

Batch Insert 는 필수다.

매우 큰 성능 저하를 유발시킬 수 있다.

> JdbcBatchInsert, Exposed 사용한다.

<br><br>

## Batch 구동 환경 (기존 스케줄 도구의 아쉬운 점)

> Jenkins, crontab, ...

### 자원 관리의 어려움

![](../../images/[Backend]%20Batch%20Performance%20극한으로%20끌어올리기_50.png)

배치 실행 시에만 자원이 사용되고 나머지 시간에는 자원이 사용되지 않는다. 

또, 특정 시간에는 배치 실행이 밀집되어 자원 경쟁이 발생할 수 있다.

<br>

### 모니터링의 어려움

![](../../images/[Backend]%20Batch%20Performance%20극한으로%20끌어올리기_17.png)

<br>

### Spring Cloud Data Flow 도입

![](../../images/[Backend]%20Batch%20Performance%20극한으로%20끌어올리기_18.png)

> 자세한 내용은 직접 찾아보자

간단하게 소개한 기능은 다음과 같다.

|기능|설명|
|-|-|
|오케스트레이션|- k8s와 완벽하게 연동되어 batch 실행을 오케스트레이션할 수 있다.<br> - 다수 Batch 가 상호 간섭 없이 동작할 수 있다. (by 컨테이너) <br>- k8s에서 resource 사용과 반납을 조율한다.|
|모니터링|- Spring Batch 와 완벽하게 호환된다. <br>- 유용한 정보를 시각적으로 모니터링할 수 있다. (Spring Cloud Date Flow 자체 Dashboard 를 제공) <br> - 그라파나 연동 가능하다.|

<br><br>

## 발표 내용 정리

![](../../images/[Backend]%20Batch%20Performance%20극한으로%20끌어올리기_33.png)