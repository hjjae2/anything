> [Kafka Connect Cluster: An Introduction](https://medium.com/clay-one/kafka-connect-cluster-an-introduction-26522e72a9af) 글을 읽고, 요약한 내용

Kafka 를 사용하다보면, 카프카 클러스터에 없는 데이터를 처리해야하는 경우가 많이 있다. 

예를 들어, 아래와 같은 곳에 존재하는 데이터를 처리해야하는 경우이다.

- DB
- 외부 파일

```
외부 스토리지   --->    Kafka 
```

```
외부 스토리지   <---    Kafka 
```

<br>


### 이를 처리하기 위한 2가지 방법이 있다.

**1. 카프카 Producer 애플리케이션을 작성한다.**

- 애플리케이션 생성
- 코드 작성
- 실패 처리 (해야한다.)
- scalability, polling

<br>

**2. Kafka Connect 를 사용한다.**

1번과 동일한 기능을 한다.

- 코드 작성 X
- 실패 처리 (해준다.)
- scalable

```
외부 스토리지   --->   Kafka Connect   --->   Kafka 
```

```
외부 스토리지   <---   Kafka Connect   <---   Kafka 
```

> 외부 저장소에서 카프카 클러스터로 데이터를 옮기는 문제가 워낙 일반적인 문제여서, Kafka Connect 가 만들어졌다고 한다.

> *" The process of copying data from a storage system and move it to Kafka Cluster is so common that Kafka Connect tool is created to address this problem. "*

<br><br>

## Kafka Connect Concepts

### worker

Kafka Connect 는 데이터를 처리(for moving data)하기 위해 `worker` 를 사용한다. <br>
- 즉, 직접적인 '처리'를 담당하는 `worker` 가 있다.
- `worker` 는 단순한 `process` 
- `worker` 는 physical concept

> *" Workers are a physical concept. They are actually processes that run inside JVM "*

`worker` 클러스터를 만들 수 있다. (= worker 여러 개를 생성 가능하다.)

`worker` 는 `worker` 의 상태, 데이터 처리(읽기) 상태 등등의 정보를 저장해야 한다.
- 이때 내부적으로 Kafka 를 사용한다. (as storage)

|MODE|설명|
|-|-|
|Distributed Mode|worker 가 여러 개 <br><br> - scalability : O <br>- fault tolerance : O|
|Standalone Mode|worker 한 개|

<br><br>

### connector

(Kafka connect 내에서) 우리들의 Job 을 `connector` 라고 부른다.

예를 들어 아래와 같은 `connector` 가 있을 수 있다.

- DB table -> Kafka cluster (`source connector`)
- Kafka cluster -> DB table (`sink connector`)

|종류|설명|
|-|-|
|source connector|외부 저장소 -> Kafka|
|sink connector|Kafka -> 외부 저장소|

<br>

많은 외부 저장소가 있기 때문에, Kafka connect 는 그 외부 저장소에 '동작 방식(읽기, 쓰기)' 을 알아야한다.

- 이를 위해 `connector plugin` 을 사용한다.

> *" Because there are many storage systems, Kafka Connect must know somehow, how to copy data from one source to Kafka or vice versa "*

<br><br>

**connector plugin**

하나 이상의 jar file 이다.

어떻게 데이터를 처리하는지(읽기, 쓰기)에 대한 방식을 아는 애다.

예를 들어 'SQL Database -> Kafka' 일 떄 `JDBC connector plugin` 을 사용할 수 

<br><br>

> 이후 설치 / 테스트하는 내용이 포함되어 있음


<br><br>

## 참고

1. [Kafka Connect Cluster: An Introduction](https://medium.com/clay-one/kafka-connect-cluster-an-introduction-26522e72a9af)