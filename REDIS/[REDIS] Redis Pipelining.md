## Redis Pipelining

> *" How to optimize round-trip times by batching Redis commands "*

여러 명령(command)을 한번에 요청/응답하는 것

> DB 에서는 Bulk 연산자를 지원하지만, 레디스에서는 Bulk 연산자를 지원하지 않는다. 대신 pipelin api 를 지원한다.

<br>

(흔히 나오는 예시) (아래)HTTP pipelining 과 비슷한 개념이다.

<img src="../images/[REDIS]%20Redis%20Pipelining_41.png" width="50%">

<br>

### 문제1. RTT (Round-Trip Time)

Redis는 고성능의 저장소이지만, TCP 기반 위에서 동작한다. 

즉 요청/응답을 위해 (TCP 기반의)네트워크 I/O가 발생할 것이다. Redis 의 성능이 아무리 좋아도, `RTT`가 길다면 클라이언트 입장에서 (시간 당)처리량이 줄어들 수 밖에 없다.

<br>

### 문제2. Socket I/O

RTT 뿐만 아니라, (파이프라이닝 없이)여러 커맨드를 요청했을 때 발생하는 socket I/O 비용도 크다. 한번에 요청하면 이 비용을 절약할 수 있다.

> *" This involves calling the `read()` and `write()` syscall, that means going from user land to kernel land. The context switch is a huge speed penalty. <br><br> When pipelining is used, many commands are usually read with a single read() system call, and multiple replies are delivered with a single write() system call. "*

<br><br>

### [Spring Data Redis :: Pipelining](https://docs.spring.io/spring-data/data-redis/docs/current/reference/html/#pipeline)

(Spring Data Redis의) `RedisTemplate` 은 파이프라이닝 기능을 지원하는 (몇 개의)메서드를 제공한다.

- `execute` / `executePipelined`

**execute**

파이프라이닝을 사용하되, 결과(result)를 신경쓰지 않는다면 `execute` 메서드에 pipeline 인자에 `true` 값만 주면 된다.

```java
public class RedisTemplate<K, V> 
        extends RedisAccessor 
        implements RedisOperations<K, V>, BeanClassLoaderAware {
    ...
    public <T> T execute(RedisCallback<T> action, boolean exposeConnection, boolean pipeline) { ... }
    ...
}
```

**executePipelined**
```java
public class RedisTemplate<K, V> 
        extends RedisAccessor 
        implements RedisOperations<K, V>, BeanClassLoaderAware {
    ...
    public List<Object> executePipelined(SessionCallback<?> session, @Nullable RedisSerializer<?> resultSerializer) {

		Assert.isTrue(initialized, "template not initialized; call afterPropertiesSet() before using it");
		Assert.notNull(session, "Callback object must not be null");

		RedisConnectionFactory factory = getRequiredConnectionFactory();
		// bind connection
		RedisConnectionUtils.bindConnection(factory, enableTransactionSupport);
		try {
			return execute((RedisCallback<List<Object>>) connection -> {
				connection.openPipeline(); // (1) : Open pipeline
				boolean pipelinedClosed = false;
				try {
					Object result = executeSession(session);
					if (result != null) {
						throw new InvalidDataAccessApiUsageException(
								"Callback cannot return a non-null value as it gets overwritten by the pipeline");
					}
					List<Object> closePipeline = connection.closePipeline(); // (3) : Close pipeline
					pipelinedClosed = true;
					return deserializeMixedResults(closePipeline, resultSerializer, hashKeySerializer, hashValueSerializer); // (2) : Deserilize (mixed)results
				} finally {
					if (!pipelinedClosed) {
						connection.closePipeline(); // (3) : Close pipeline
					}
				}
			});
		} finally {
			RedisConnectionUtils.unbindConnection(factory);
		}
	}

    ...

    public List<Object> executePipelined(RedisCallback<?> action, @Nullable RedisSerializer<?> resultSerializer) { ... }
    ...
}
```

> pipeline 을 열고, 닫는것을 제외하고는 거의 execute 와 유사하다.

<br>

**예시**

```java
List<Object> results = stringRedisTemplate.executePipelined(
  new RedisCallback<Object>() {
    public Object doInRedis(RedisConnection connection) throws DataAccessException {
      StringRedisConnection stringRedisConn = (StringRedisConnection)connection;
      for(int i=0; i< batchSize; i++) {
        stringRedisConn.rPop("myqueue");
      }
    return null;
  }
});
```

> 위 예시는 하나의 요청(커넥션)에서 bulk rPop 연산을 처리하는 예시이다.
> 
> `results` 에는 pop 된 아이템들이 결과물로 나온다.

<br><br>

### RedisConnection.openPipeline()

```java
public interface RedisConnection extends RedisCommands, AutoCloseable {
    ...

    /**
	 * Activates the pipeline mode for this connection. When pipelined, all commands return null (the reply is read at the
	 * end through {@link #closePipeline()}. Calling this method when the connection is already pipelined has no effect.
	 * Pipelining is used for issuing commands without requesting the response right away but rather at the end of the
	 * batch. While somewhat similar to MULTI, pipelining does not guarantee atomicity - it only tries to improve
	 * performance when issuing a lot of commands (such as in batching scenarios).
	 * <p>
	 * Note:
	 * </p>
	 * Consider doing some performance testing before using this feature since in many cases the performance benefits are
	 * minimal yet the impact on usage are not.
	 *
	 * @see #multi()
	 */
	void openPipeline();

    ...
}
```

```java
public class LettuceConnection extends AbstractRedisConnection {

    ...

    @Override
	public void openPipeline() {
		if (!isPipelined) {
			isPipelined = true;
			ppline = new ArrayList<>();
			flushState = this.pipeliningFlushPolicy.newPipeline();
			flushState.onOpen(this.getOrCreateDedicatedConnection());
		}
	}

    ...
}
```

특징을 살펴보면, 

1. (해당 커넥션에) 파이프라인 모드를 활성화한다.
   - 이미 파이프라인이 오픈된 상태에서는 아무 효과가 없다.
2. 파이프라인 동안에는, 모든 명령어는 null 을 반환한다. (null 반환하지 않을 경우 Exception 터지는 로직이 있기도 함)
3. MULTI 와 비슷할 수 있지만, pipelining 은 atomicity 를 보장하지는 않는다.
   - 파이프라이닝은 다수의 명령어에 대한 성능 향상이 목적이다.


<br><br>

### RedisConnection.closePipeline()

```java
public interface RedisConnection extends RedisCommands, AutoCloseable {
    ...

    /**
	 * Executes the commands in the pipeline and returns their result. If the connection is not pipelined, an empty
	 * collection is returned.
	 *
	 * @throws RedisPipelineException if the pipeline contains any incorrect/invalid statements
	 * @return the result of the executed commands.
	 */
	List<Object> closePipeline() throws RedisPipelineException;

    ...
}
```

```java
public class LettuceConnection extends AbstractRedisConnection {

    ...

    @Override
	public List<Object> closePipeline() {

		if (!isPipelined) {
			return Collections.emptyList();
		}

		flushState.onClose(this.getOrCreateDedicatedConnection());
		flushState = null;
		isPipelined = false;
		List<io.lettuce.core.protocol.RedisCommand<?, ?, ?>> futures = new ArrayList<>(ppline.size());
		for (LettuceResult<?, ?> result : ppline) {
			futures.add(result.getResultHolder());
		}

		try {
			boolean done = LettuceFutures.awaitAll(timeout, TimeUnit.MILLISECONDS,
					futures.toArray(new RedisFuture[futures.size()]));

			List<Object> results = new ArrayList<>(futures.size());

			Exception problem = null;

			if (done) {
				for (LettuceResult<?, ?> result : ppline) {

					if (result.getResultHolder().getOutput().hasError()) {

						Exception err = new InvalidDataAccessApiUsageException(result.getResultHolder().getOutput().getError());
						// remember only the first error
						if (problem == null) {
							problem = err;
						}
						results.add(err);
					} else if (!result.isStatus()) {

						try {
							results.add(result.conversionRequired() ? result.convert(result.get()) : result.get());
						} catch (DataAccessException e) {
							if (problem == null) {
								problem = e;
							}
							results.add(e);
						}
					}
				}
			}
			ppline.clear();

			if (problem != null) {
				throw new RedisPipelineException(problem, results);
			}

			if (done) {
				return results;
			}

			throw new RedisPipelineException(new QueryTimeoutException("Redis command timed out"));
		} catch (Exception e) {
			throw new RedisPipelineException(e);
		}
	}

    ...
}
```

특징을 살펴보면,
1. 파이프라인 안에서 실행한 명령어들에 대한 결과(results)가 반환된다.
   - 파이프라이닝 상태가 아니었다면, 빈 컬렉션(empty collection)이 반환된다.

<br><br>

### 추가

**1. command 의 순서가 변경되지 않는다.**

Redis pipelining 은 단순히 요청을 한번에 보내는 것일 뿐 요청(커맨드)의 순서에 영향을 주지 않는다. <br>
공식 문서에서 이것과 관련된 내용은 못찾았다. 다만, [여기](https://stackoverflow.com/questions/17634826/redis-pipelined-order-of-execution)를 포함해 다른 글들을 참고할 수 있을 것 같다.

<br><br>

### 출처

- [Redis pipelining](https://redis.io/topics/pipelining)
- [Spring Data Redis](https://docs.spring.io/spring-data/data-redis/docs/current/reference/html/#pipeline)