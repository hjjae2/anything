### TCP 연결을 맺고 동시에 여러 요청(API 호출)을 했을 때 :arrow_right: 서로 다른 Thread 로 처리

<img src="/images/spring/tcp,thread 비교.png" width="100%" height="100%">


<br>

### 내장 톰캣 설정 확인

`org.springframework.boot.autoconfigure.web.ServerProperties`

```java
package org.springframework.boot.autoconfigure.web;

@ConfigurationProperties(prefix = "server", ignoreUnknownFields = true)
public class ServerProperties {

	...


	/**
	 * Tomcat properties.
	 */
	public static class Tomcat {

		/**
		 * Maximum number of connections that the server accepts and processes at any
		 * given time. Once the limit has been reached, the operating system may still
		 * accept connections based on the "acceptCount" property.
		 */
		private int maxConnections = 8192;

		/**
		 * Maximum queue length for incoming connection requests when all possible request
		 * processing threads are in use.
		 */
		private int acceptCount = 100;

		/**
		 * Maximum number of idle processors that will be retained in the cache and reused
		 * with a subsequent request. When set to -1 the cache will be unlimited with a
		 * theoretical maximum size equal to the maximum number of connections.
		 */
		private int processorCache = 200;

		...

	}

	...

	/**
		* Tomcat thread properties.
		*/
	public static class Threads {

		/**
			* Maximum amount of worker threads.
			*/
		private int max = 200;

		/**
			* Minimum amount of worker threads.
			*/
		private int minSpare = 10;

		...

	}
```

<br><br>

(위의 설정을 기준으로)

1. 우선, 10개의 thread 로 처리
2. 요청이 왔는데 idle 상태의 thread 가 없다면, 새로운 Thread 생성 -> 이게 max(200)값 까지 가능 <br>
\* 일단 minSpare 보다 더 생기면, 언제 없어지는지 확인 필요

```java
// 예시 로그 1
[2022-03-06 18:47:30:6150] | [http-nio-8080-exec-1] | [DEBUG] o.a.coyote.http11.Http11NioProtocol - Created new processor [org.apache.coyote.http11.Http11Processor@3adee237]

[2022-03-06 18:47:50:26136] | [http-nio-8080-exec-2] | [DEBUG] o.a.coyote.http11.Http11NioProtocol - Created new processor [org.apache.coyote.http11.Http11Processor@3da5aceb]

[2022-03-06 18:48:52:88219] | [http-nio-8080-exec-2] | [DEBUG] o.a.coyote.http11.Http11NioProtocol - Processing socket [org.apache.tomcat.util.net.NioChannel@336254f7:java.nio.channels.SocketChannel[connected local=/0:0:0:0:0:0:0:1:8080 remote=/0:0:0:0:0:0:0:1:65371]] with status [ERROR]
[2022-03-06 18:48:52:88219] | [http-nio-8080-exec-2] | [DEBUG] o.a.coyote.http11.Http11NioProtocol - Found processor [null] for socket [org.apache.tomcat.util.net.NioChannel@336254f7:java.nio.channels.SocketChannel[connected local=/0:0:0:0:0:0:0:1:8080 remote=/0:0:0:0:0:0:0:1:65371]]
[2022-03-06 18:48:52:88219] | [http-nio-8080-exec-2] | [DEBUG] o.a.tomcat.util.threads.LimitLatch - Counting down[http-nio-8080-exec-2] latch=1
```



```java
// 예시 로그 2
[2022-03-06 18:43:15:31335] | [http-nio-8080-exec-2] | [DEBUG] o.a.coyote.http11.Http11NioProtocol - Processing socket [org.apache.tomcat.util.net.NioChannel@693ffb87:java.nio.channels.SocketChannel[connected local=/0:0:0:0:0:0:0:1:8080 remote=/0:0:0:0:0:0:0:1:64227]] with status [OPEN_READ]
[2022-03-06 18:43:15:31335] | [http-nio-8080-exec-2] | [DEBUG] o.a.coyote.http11.Http11NioProtocol - Found processor [null] for socket [org.apache.tomcat.util.net.NioChannel@693ffb87:java.nio.channels.SocketChannel[connected local=/0:0:0:0:0:0:0:1:8080 remote=/0:0:0:0:0:0:0:1:64227]]
[2022-03-06 18:43:15:31335] | [http-nio-8080-exec-2] | [DEBUG] o.a.coyote.http11.Http11NioProtocol - Popped processor [null] from cache
[2022-03-06 18:43:15:31335] | [http-nio-8080-exec-2] | [DEBUG] o.a.coyote.http11.Http11NioProtocol - Register [org.apache.coyote.http11.Http11Processor@67f808d0] as [Tomcat:type=RequestProcessor,worker="http-nio-8080",name=HttpRequest2]
[2022-03-06 18:43:15:31335] | [http-nio-8080-exec-2] | [DEBUG] o.a.coyote.http11.Http11NioProtocol - Created new processor [org.apache.coyote.http11.Http11Processor@67f808d0]
[2022-03-06 18:43:15:31336] | [http-nio-8080-exec-2] | [DEBUG] o.a.coyote.http11.Http11InputBuffer - Before fill(): parsingHeader: [true], parsingRequestLine: [true], parsingRequestLinePhase: [0], parsingRequestLineStart: [0], byteBuffer.position(): [0], byteBuffer.limit(): [0], end: [0]
[2022-03-06 18:43:15:31336] | [http-nio-8080-exec-2] | [DEBUG] o.a.t.util.net.SocketWrapperBase - Socket: [org.apache.tomcat.util.net.NioEndpoint$NioSocketWrapper@63c790f7:org.apache.tomcat.util.net.NioChannel@693ffb87:java.nio.channels.SocketChannel[connected local=/0:0:0:0:0:0:0:1:8080 remote=/0:0:0:0:0:0:0:1:64227]], Read from buffer: [0]
[2022-03-06 18:43:15:31336] | [http-nio-8080-exec-2] | [DEBUG] o.a.tomcat.util.net.NioEndpoint - Socket: [org.apache.tomcat.util.net.NioEndpoint$NioSocketWrapper@63c790f7:org.apache.tomcat.util.net.NioChannel@693ffb87:java.nio.channels.SocketChannel[connected local=/0:0:0:0:0:0:0:1:8080 remote=/0:0:0:0:0:0:0:1:64227]], Read direct from socket: [869]
[2022-03-06 18:43:15:31336] | [http-nio-8080-exec-2] | [DEBUG] o.a.coyote.http11.Http11InputBuffer - Received [GET /test HTTP/1.1
```


<br>

**설정 예시**
```yaml
server:
  tomcat:
    threads:
      max: 1
      min-spare: 1
    accept-count: 1
```

<br>

### 참고하기

1. https://velog.io/@sihyung92/how-does-springboot-handle-multiple-requests