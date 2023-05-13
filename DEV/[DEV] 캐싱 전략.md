> [6-Caching Strategies to Remember while designing Cache System](https://medium.com/javascript-in-plain-english/6-caching-strategies-to-remember-while-designing-cache-system-da058a3757cf)

## Key Metrics

1. 캐시 히트율 (cache hit ratio)
2. 응답 시간 (latency)
3. 처리량 (throughput)
4. 캐시 사이즈 (cache size)
5. 캐시 미스율 (cache miss ratio)

<br>

## Read Intensive Application Caching

1. Cache-aside
2. Cache-through
3. Refresh-ahead

### Cache-aside

![](../images/[DEV]%20캐싱%20전략_00.png)

1. 캐시가 미스됐을 때, 3번의 trip 이 발생해 급격한 딜레이가 생길 수 있다.
2. 데이터베이스에 업데이트하고 캐시에는 업데이트를 누락하여 캐시의 데이터가 낡은 상태로 유지될 수 있다.

 > *" Data might become stale if someone updates the database without writing to the cache. (Hence, Cache-aside is usually used alongside other strategies). "*

### Read-through

![](../images/[DEV]%20캐싱%20전략_48.png)

Cache-aside 와 비슷하지만, 애플리케이션이 캐시에만 질의하고, (캐시 미스 상태인 경우) 캐시가 DB에 질의하는 차이가 있다.

1. Read-through 는 (애플리케이션 -> 캐시 -> DB)흐름/로직이 단순해진다.
2. Better read scalability. Cache-aside 에서 키가 만료된 경우, 동시 요청이 발생해 DB에 여러번 질의할 수 있다. 그러나 Read-through의 경우, (동시 요청이 발생해도)DB에는 오직 1번의 쿼리만 질의되는 것이 보장된다. 
> (아마도 흐름은...) 
> 1. 캐시에 동시 여러개의 요청이 왔을 때 그 중 하나의 처리에서 업데이트(DB로부터 데이터 질의 & 업데이트) 
> 2. 이때 나머지 요청은 블락(큐잉) 상태

### Refersh-ahead

![](../images/[DEV]%20캐싱%20전략_35.png)

> *" So what refresh ahead caching does is it essentially refreshes the cache at a configured interval just before the next possible cache access although it might take some time due to network latency to refresh the data & meanwhile few thousand read operation already might have happened in a very highly read heavy system in just a duration of few milliseconds. "*

<br>

## Write Intensive Application Caching

1. Write-through
2. Write-back (Write-behind)

### Write-through

![](../images/[DEV]%20캐싱%20전략_13.png)

> " The Write-Through strategy treats cache as **its primary data store**, that is , the data is updated first in cache and then in database. "

Read-through 와 비슷하다. Write-through 는 캐시가 primary data store 로 취급되는 것과 마찬가지다.

### Write-back

![](../images/[DEV]%20캐싱%20전략_22.png)