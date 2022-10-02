### indexing (색인)

오리지널 데이터(원본 데이터)를 **'데이터가 검색될 수 있는 구조로 변경하기 위해'** 토큰화하거나 변환하는 과정.

<br>

### index (인덱스)

indexing 과정을 거친 후의 결과물.

(또는) 색인된 데이터가 저장되는 저장소.

> ElasticSearch 에서는 Document 들의 논리적인 집합, 표현 단위이기도 하다.

<br>

### search (검색)

인덱스에 들어있는 데이터를 찾는 것.

> 혹은 인덱스된 데이터를 찾는 것

<br>

### query (질의)

search 를 하기 위한 입력어.

<br>

### 간단한 구조

```sh
                  indexing                          search
original data     -------->     index (indicies)    ------->    result
```
