프로메테우스는 실시간으로 시계열 데이터를 선택해 집계할 수 있는 PromQL(함수형 쿼리 언어)를 제공한다.

## [Type (Expression Language Data Types)](https://godekdls.github.io/Prometheus/querying.basics/)

프로메테우스의 표현식 언어에서, 표현식 또는 하위 표현식은 다음 타입 중 하나로 평가될 수 있다.

- **Instant vector** : 같은 타임스탬프 상에 있는 시계열 셋으로, 각 시계열마다 단일 샘플을 가지고 있다.
- **Range vector** : 특정 시간 범위에 있는 시계열 셋으로, 각 시계열마다 시간에 따른 데이터 포인트들을 가지고 있다.
- **Scalar** : 간단한 부동 소수점 숫자
- **String** : (현재 사용 X) 간단한 문자열 값

표현식을 사용하는 방식(e.g., 그래프 표현, 데이터 출력 등)에 따라 정의할 수 있는 타입은 제한돼있다.

<br>

### Instant Vector Selectors

Instant Vector (셀렉터)를 사용하면 지정한 타임스탬프(instant)에서 시계열 셋을 선택해, 각 시계열마다 단일 샘플 값을 가져올 수 있다. 

가장 간단하게는 메트릭 이름만 지정할 수 있다. 메트릭명을 가진 모든 시계열 요소들을 갖고 있는 instant 벡터가 생성된다.

**예시 - `http_requests_total`**

```promql
http_requests_total{job="~~~",group="~~~"}
```

http_requests_total 시계열 중에서 job, group에 매칭되는 모든 시계열을 가져온다.

<br>

### Range Vector Selectors

Range Vector (셀렉터)는 range(범위)의 샘플을 가져온다는 점을 제외하고 instant vector 와 유사하게 동작한다.

Range Vector Selector 의 경우 끝에 대괄호`[]`를 사용해 범위(range)를 추가한다.


**예시 - `http_requests_total`**

```promql
http_requests_total{job="~~~",group="~~~"}[5m]
```
http_requests_total 시계열 중에서 job, group에 매칭되는 **현재 ~ 지난 5분 이내의** 모든 시계열을 가져온다.

<br><br>

## Functions

### idelta()

idelta(v ragne-vector)는 (range 벡터)v에 있는 마지막 두 샘플의 차이를 계산하고, 계산한 델타 값을 레이블과 함께 instant 벡터로 반환한다.

idelta 에선 게이지(gauge)만 사용해야 한다.

<br>

### increase()

**increase(v range-vector)는 range 벡터 안에 있는 시계열의 증분을 계산한다.**

range 벡터 셀렉터에 지정된 시간 범위를 커버하기 위해 어림잡아 증분을 계산한다. 때문에, 실제 결과 값에 차이가 있을 수 있다. 예를 들어, 카운터 값이 정수 단위로만 증가하더라도 결과는 정수가 아닐 수 있다.

```promql
// 지난 5분 동안의 http 요청 수를 반환한다.
increase(http_requests_total{job="~~~"}[5m])
```

increase 는 카운터에만 사용해야 한다. 

increase 는 단순히 rate(v) 함수에 지정한 time range 만큼 초(sec)를 곱한 함수일 뿐이다. 

**주로 가독성이 요구될 때만 사용하는 게 좋다. recording rule 에는 증분을 지속적으로 초 단위로 추적할 수 있도록 rate 를 사용하는 것이 좋다.**

<br>

### irate()

irate(v range-vector)는 range 벡터 안에 있는 시계열의 **초당 순간 변화율**을 계산한다. 변화율은 마지막 데이터 포인트 2개를 사용해 계산한다.

```promql
// 지난 5분 동안의 http 요청 수를 조회하고, 가장 최근 데이터 포인트 2개를 통해 초당 HTTP 요청 비율을 계산한다.
irate(http_requests_total{job="~~~"}[5m])
```

irate 는 변덕스럽고 빠르게 변화하는 카운터를 그래프로 표현할 때만 사용하는 것이 좋다. **alert 혹은 느리게 변화하는 카운터에는 rate를 사용하는 것이 좋다.** irate 는 비율이 잠깐 변경되어도 FOR 절을 리셋할 수 있고, 그래프 전체를 드문드문 스파이크로 만들 수 있기 때문이다. (= 보기 어렵게 한다.)

<br>

### rate()

rate(v rage-vector)는 range 벡터 안에 있는 시계열의 **초당 평균 변화율**을 계산한다. 

> 시간 범위의 양 끝은 어림잡아 계산한다. 때문에, 스크랩 일부가 누락되어도 문제가 없으며, range 의 기간과 스크랩 주기가 정확히 일렬로 정렬되지 않아도 괜찮다.

```promql
// 지난 5분 동안 초당 HTTP 요청 비율
rate(http_requests_total{job="~~~"}[5m])
```

rate 는 카운터에만 사용해야 한다. **alert 정의, 느리게 변화하는 카운터를 그래프로 표현할 땐 rate가 가장 적합하다.**

<br><br>

## rate vs increase

|rate|increase|
|-|-|
|범위 시간당 초당 호출량|범위 시간당 증가율|

### rate

rate 는 **초당** 호출량을 특정 기간을 기준으로 측정하는 것이다.

만약 1분 동안 호출이 60번 됐다면 (1tps), 아래 쿼리의 결과는 1이다.

```promql
rate(http_request_count_total[1m]) // 1
```

**tps 를 원할 때, rate 를 사용하면 되겠다.**

> 기준이 **초** 라는 것이 중요하다.

<br>

### increase

호출 횟수를 그래프로 그리고, 표현하고 싶을 때는 increase를 사용하면 된다. 

1분 동안 호출이 60번 됐다면, 아래 쿼리의 결과는 60이다.

```promql
increase(http_request_count_total[1m])
```

> 단순히 range 동안 증분한 결과로 보면 되겠다.

**increase 를 사용해 그래프를 표현할 때 `Min time interval`을 단위 시간(위에서는 1m)과 동일하게 맞춰줘야 한다.** 그래야 단위 시간 기준으로 정확한 데이터(min, max, avg)를 얻을 수 있다.

<br><br>

## 참고

- https://blog.voidmainvoid.net/449
- https://godekdls.github.io/Prometheus