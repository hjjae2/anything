## Channel

> *" A nexus to a network socket or a component which is capable of I/O operations such as read, write, connect, and bind. "*

**I/O(read, write, connect, bind) 작업을 할 수 있는 네트워크 소켓, 컴포넌트와의 연결점이다.**

<br>

**다음 기능을 제공한다.**

- channel 의 현재 상태
- channel 의 구성 파라미터 (configuration parameters)
- channel 이 지원하는 I/O 작업 (read, write, connect, bind)
- channel 과 관련된 모든 I/O 이벤트, requests 를 핸들링하는 ChannelPipeline

<br>

**모든 I/O 작업은 비동기이다.**

따라서, channel 의 I/O 작업은 즉시 응답을 return 한다. 

- 작업의 성공 여부를 확인할 수 있는 `ChannelFuture` 객체를 반환한다.
- (당연하게도) I/O 작업이 성공했다거나, 완료되었음을 보장하지는 않는다.

<br>

**(사용 후에)`close()`, `close(ChannelPromise)`를 호출하는 것이 중요하다.**

channel 사용이 완료되면, 모든 resource 를 해제(release)하기 위해 `close()`, `close(ChannelPromise)` 를 호출하는 것이 중요하다.

모든 리소스는 적절한 방식으로 해제(release)된다.

> *" It is important to call close() or close(ChannelPromise) to release all resources once you are done with the Channel. This ensures all resources are released in a proper way, i.e. filehandles. "*

```java
package io.netty.channel;

...

public interface Channel extends AttributeMap, ChannelOutboundInvoker, Comparable<Channel> {

    /**
     * Returns the globally unique identifier of this {@link Channel}.
     */
    ChannelId id();

    ...
```

<br><br>

## ChannelFuture

**Channel I/O 비동기 작업의 결과이다.**

<br>

**ChannelFuture 는 `completed`, `uncompleted` 둘 중 하나의 상태이다.**

1. I/O 작업이 시작될 때, 하나의 `ChannelFuture` 객체가 생성된다. (`uncompleted`)
   1. I/O 작업이 완료되기 전까지, 성공, 실패, 취소 모두 아니다.
2. I/O 작업이 끝났다면, 해당 상태(성공, 실패, 취소)로 마킹된다.
   1. '실패', '취소'도 완료 상태에 속한다는 것을 유의해야 한다.


```text
                                       +---------------------------+
                                       | Completed successfully    |
                                       +---------------------------+
                                  +---->      isDone() = true      |
  +--------------------------+    |    |   isSuccess() = true      |
  |        Uncompleted       |    |    +===========================+
  +--------------------------+    |    | Completed with failure    |
  |      isDone() = false    |    |    +---------------------------+
  |   isSuccess() = false    |----+---->      isDone() = true      |
  | isCancelled() = false    |    |    |       cause() = non-null  |
  |       cause() = null     |    |    +===========================+
  +--------------------------+    |    | Completed by cancellation |
                                  |    +---------------------------+
                                  +---->      isDone() = true      |
                                       | isCancelled() = true      |
                                       +---------------------------+
  
```

위 예시처럼(?) 상태(성공, 실패, 취소)를 확인할 수 있는 다양한 메서드를 제공한다.

<br>

**`ChannelFutureListener` 인터페이스를 통해 리스너 구현체를 손쉽게 작성할 수 있다.** <br>

I/O 작업이 완료되었을 때의 이벤트를 수신한다.

await 보다 addListener 를 권장한다.

- await : blocking
  - (thread)dead lock 발생할 수 있다.
- addListener : non-blocking

> *" Prefer addListener(GenericFutureListener) to await()  <br>(중략...)<br> Moreover, there's a chance of dead lock in a particular circumstance, which is described below. <br>(중략...)<br> Do not call await() inside ChannelHandler "*

> await 가 분명히 편리한 경우도 있다. 그럼에도 I/O thread 에서는 호출하지 마라. 

> *" In spite of the disadvantages mentioned above, there are certainly the cases where it is more convenient to call await(). In such a case, please make sure you do not call await() in an I/O thread. Otherwise, BlockingOperationException will be raised to prevent a dead lock. "*

<br>

**'I/O 타임아웃'과 'await 타임아웃' 을 혼동하지 않아야 한다.**

> *" Do not confuse I/O timeout and await timeout "*

아래와 같이 await 함수 사용 시 타임아웃을 제한할 수 있다.

- `await(long)`
- `await(long, TimeUnit)`
- `awaitUninterruptibly(long)`
- `awaitUninterruptibly(long, TimeUnit)` 

다만, 이 타임아웃은 I/O 타임아웃과는 전혀 관련이 없다.

> 이 부분은 실제 사용 시 유의해야할 것 같다. (실제 사용 시 한번 더 이해가 필요하다.)

<br><br>

## ChannelHandler

I/O 이벤트를 처리(핸들링)하거나, I/O 작업을 가로챈다.(intercept) <br>
그리고 (이벤트)를 (ChannelPipeline 의) 다음 핸들러로 전달한다.

<br>

**ChannelHandler는 많은 메서드를 제공하지는 않지만, 일반적으로 다음 하위 타입을 구현해야한다.**

- `ChannelInboundHandler` : To handle inbound I/O events
- `ChannelOutboundHandler` : To handle outbound I/O operations

<br>

**(대안으로) 편의를 위해 다음과 같은 adapter 클래스도 제공한다.**

- `ChannelInboundHandlerAdapter` : To handle inbound I/O events
- `ChannelOutboundHandlerAdapter` : To handle outbound I/O operations
- `ChannelDuplexHandler` : To handle both inbound and outbound events

<br>

**Context Object**

`ChannelHandler` 는 `ChannelHandlerContext` 객체와 함께 제공된다.

하나의 `ChannelHandler`는 이 contect 객체(`ChannelHandlerContext`)를 통해 자신이 속한 `ChannelPipeline` 과 상호작용해야 한다.

Contect 객체를 사용해서 다음과 같은 작업을 할 수 있다.

1. 업스트림, 다운스트림에 이벤트를 전달할 수 있다.
2. Pipeline 을 동적으로 수정할 수 있다.
3. 핸들러에 특정한 정보를 저장할 수 있다. (using `AttributeKeys`)

> *" A ChannelHandler is provided with a ChannelHandlerContext object. A ChannelHandler is supposed to interact with the ChannelPipeline it belongs to via a context object. Using the context object, the ChannelHandler can pass events upstream or downstream, modify the pipeline dynamically, or store the information (using AttributeKeys) which is specific to the handler. "*

<br>

**상태 관리(State Management)**

`ChannelHandler`는 종종 stateful information 을 저장해야 할 필요/니즈가 있다.

가장 쉽고, 추천하는 방법은 `멤버 변수` 를 사용하는 것이다.

아래 예시는 `DataServerHandler` 라는 핸들러에서 `loggedIn` 이라는 멤버 변수를 사용하는 예시이다.

```java
  public class DataServerHandler extends SimpleChannelInboundHandler<Message> {
 
      private boolean loggedIn;
 
      @Override
      public void channelRead0(ChannelHandlerContext ctx, Message message) {
          if (message instanceof LoginMessage) {
              authenticate((LoginMessage) message);
              loggedIn = true; // 여기! (stateful information)
          } else (message instanceof GetDataMessage) {
              if (loggedIn) { // 여기! (stateful information)
                  ctx.writeAndFlush(fetchSecret((GetDataMessage) message));
              } else {
                  fail();
              }
          }
      }
      ...
  }
```

다만 위와 같이 상태를 갖게한다면, 커넥션 별로 별도의 handler 객체를 생성해야 한다.

1. Handler 객체는 하나의 커넥션(one connection)에 전용인 state 변수를 갖는다. 
2. 각각의 커넥션 별로 새로운 핸들러 객체를 만들어야 한다. (race condition 을 피하기 위해서)

아래 예시는 Channel 이니셜라이저에 handler 를 등록하는 것을 보여주는 예시같다. <br>
= 채널마다 새로운 핸들러 객체를 생성하는 것을 보여주는 것 같다.

```java
  
  // Create a new handler instance per channel.
  // See ChannelInitializer.initChannel(Channel).
  public class DataServerInitializer extends ChannelInitializer<Channel> {
      @Override
      public void initChannel(Channel channel) {
          channel.pipeline().addLast("handler", new DataServerHandler());
      }
  }
```
> 일반적인 상황에서도 별도의 객체를 생성하는 것을 말하는 건지, 위와 같이 멤버 변수(상태 변수)를 가질 때에만을 말하는 건지 헷갈린다.

<br>

**멤버 변수 사용을 원치 않는다면, `AttributeKeys` 를 사용할 수 있다.**

> `ChannelHandlerContext` 에 의해 제공된다.

```java
  @Sharable // 밑에서 살펴볼 어노테이션이다.
  public class DataServerHandler extends SimpleChannelInboundHandler<Message> {
      private final AttributeKey<Boolean> auth =
            AttributeKey.valueOf("auth"); // 1. AttributeKey 를 만든다.
 
      @Override
      public void channelRead(ChannelHandlerContext ctx, Message message) {
          Attribute<Boolean> attr = ctx.attr(auth); // 2. ChannelHandlerContext 에서 attribute 를 가져온다.
          if (message instanceof LoginMessage) {
              authenticate((LoginMessage) o);
              attr.set(true); // 3-1. attr 값을 설정한다. (여기서는 boolean 타입이다. true 로 설정한다.)
          } else (message instanceof GetDataMessage) {
              if (Boolean.TRUE.equals(attr.get())) { // 3-1. attr 값을 가져온다. (여기서는 boolean 타입이다.)
                  ctx.writeAndFlush(fetchSecret((GetDataMessage) o));
              } else {
                  fail();
              }
          }
      }
      ...
  }
```

<br>

**`@Sharable`**

Handler 객체를 한 번만 생성할 수 있음을 표시한다.

- Race Condition 없이 여러 개의 ChannelPipeline 에 추가/등록할 수 있음을 표시한다.

이 주석이 없다면 공유되어 사용될 수 없는(race condition 이 발생할 수 있는) 멤버 변수를 갖고 있다는 뜻으로 간주하고, 매 번 새로운 객체를 생성해야 한다.

<br>

```java
package io.netty.channel;

public interface ChannelHandler {

    /**
     * Gets called after the {@link ChannelHandler} was added to the actual context and it's ready to handle events.
     */
    void handlerAdded(ChannelHandlerContext ctx) throws Exception;

    ...

}
```

<br><br>

## ChannelPipeline

ChannelHandler 들의 리스트이다.

> *" A list of ChannelHandlers which handles or intercepts inbound events and outbound operations of a Channel. "*

ChannelPipeline은 이벤트가 처리되는 방식과 파이프라인의 ChannelHandler가 서로 상호 작용하는 방식을 사용자에게 완전히 제어할 수 있도록 Intercepting Filter 패턴의 고급 형태를 구현합니다.

(하나의 Pipeline 안에서) 하나의 Event 가 처리되는 방식, ChannelHandler 들이 서로 상호 작용하는 방식에 대한 정보를 갖고 있는 클래스라고 보면 될 것 같다.

> Event 를 처리하는 방식, Handler들을 적용하는 방식을 구현한다.
> 
> 또, 사용자가 이것들을 완전히 컨트롤할 수 있게 해준다.
> 
> 약간 FilterChainProxy 같은 느낌일 것 같다.

<br>

**파이프라인 생성 (Creation of a pipeline)**

각각의 channel 은 자신만의 파이프라인을 갖는다. (파이프라인에 속하는 느낌인 것 같다.)
= 채널이 생성될 때 자동으로 파이프라인이 생성된다. (?)

> *" Each channel has its own pipeline and it is created automatically when a new channel is created. "*

<br>

**하나의 파이프라인에서 Event Flow (How an event flows in a pipeline)**

```text
                                                  I/O Request
                                             via Channel or
                                         ChannelHandlerContext
                                                       |
   +---------------------------------------------------+---------------+
   |                           ChannelPipeline         |               |
   |                                                  \|/              |
   |    +---------------------+            +-----------+----------+    |
   |    | Inbound Handler  N  |            | Outbound Handler  1  |    |
   |    +----------+----------+            +-----------+----------+    |
   |              /|\                                  |               |
   |               |                                  \|/              |
   |    +----------+----------+            +-----------+----------+    |
   |    | Inbound Handler N-1 |            | Outbound Handler  2  |    |
   |    +----------+----------+            +-----------+----------+    |
   |              /|\                                  .               |
   |               .                                   .               |
   | ChannelHandlerContext.fireIN_EVT() ChannelHandlerContext.OUT_EVT()|
   |        [ method call]                       [method call]         |
   |               .                                   .               |
   |               .                                  \|/              |
   |    +----------+----------+            +-----------+----------+    |
   |    | Inbound Handler  2  |            | Outbound Handler M-1 |    |
   |    +----------+----------+            +-----------+----------+    |
   |              /|\                                  |               |
   |               |                                  \|/              |
   |    +----------+----------+            +-----------+----------+    |
   |    | Inbound Handler  1  |            | Outbound Handler  M  |    |
   |    +----------+----------+            +-----------+----------+    |
   |              /|\                                  |               |
   +---------------+-----------------------------------+---------------+
                   |                                  \|/
   +---------------+-----------------------------------+---------------+
   |               |                                   |               |
   |       [ Socket.read() ]                    [ Socket.write() ]     |
   |                                                                   |
   |  Netty Internal I/O Threads (Transport Implementation)            |
   +-------------------------------------------------------------------+
```

> Spring MVC 에서 Filter, Intercept 가 처리되는 방식과 유사하다.

하나의 이벤트는 (위에서 구현해야한다고 했던)`ChannelInboundHandler`, `ChannelOutboundHandler` 구현체들에 의해 처리된다.

그림에서도 볼 수 있듯이, 아웃바운드 핸들러를 모두 통과하고 나면 해당 channel 과 연결되어 있는 I/O thread 에 의해 처리된다.

I/O thread 는 실제 작업(write)을 수행한다.

> 인바운드 데이터는 대부분 원격지로부터 읽어온다.
>  
> " The inbound data is often read from a remote peer via the actual input operation such as SocketChannel.read(ByteBuffer). "

<br>

**Order of Handlers**

아래와 같은 핸들러들이 있다고 가정하자.

```java
  ChannelPipeline p = ...;
  p.addLast("1", new InboundHandlerA());
  p.addLast("2", new InboundHandlerB());
  p.addLast("3", new OutboundHandlerA());
  p.addLast("4", new OutboundHandlerB());
  p.addLast("5", new InboundOutboundHandlerX());
```

인바운드일 때, 핸들러 적용 순서는 다음과 같다.

- 1 → 2 → 3 → 4 → 5
- 1 → 2 → 5

아웃바운드일 때, 핸들러 적용 순서는 다음과 같다.

- 5 → 4 → 3 → 2 → 1
- 5 → 4 → 3

<br>

**Forwarding an event to the next handler**

Handler 가 다음 Handler 로 이벤트를 전파하기 위해서 다음과 같은 메서드를 사용한다.

Inbound event propagation methods:

- ChannelHandlerContext.fireChannelRegistered()
- ChannelHandlerContext.fireChannelActive()
- ChannelHandlerContext.fireChannelRead(Object)
- ChannelHandlerContext.fireChannelReadComplete()
- ChannelHandlerContext.fireExceptionCaught(Throwable)
- ChannelHandlerContext.fireUserEventTriggered(Object)
- ChannelHandlerContext.fireChannelWritabilityChanged()
- ChannelHandlerContext.fireChannelInactive()
- ChannelHandlerContext.fireChannelUnregistered()

Outbound event propagation methods:

- ChannelHandlerContext.bind(SocketAddress, ChannelPromise)
- ChannelHandlerContext.connect(SocketAddress, SocketAddress, ChannelPromise)
- ChannelHandlerContext.write(Object, ChannelPromise)
- ChannelHandlerContext.flush()
- ChannelHandlerContext.read()
- ChannelHandlerContext.disconnect(ChannelPromise)
- ChannelHandlerContext.close(ChannelPromise)
- ChannelHandlerContext.deregister(ChannelPromise)

사용 예시는 다음과 같다.

```java
  public class MyInboundHandler extends ChannelInboundHandlerAdapter {
      @Override
      public void channelActive(ChannelHandlerContext ctx) {
          System.out.println("Connected!");
          ctx.fireChannelActive();
      }
  }
 
  public class MyOutboundHandler extends ChannelOutboundHandlerAdapter {
      @Override
      public void close(ChannelHandlerContext ctx, ChannelPromise promise) {
          System.out.println("Closing ..");
          ctx.close(promise);
      }
  }
```

<br>

**Building a pipeline**

> *" Be aware that while using DefaultEventLoopGroup will offload the operation from the EventLoop it will still process tasks in a serial fashion per ChannelHandlerContext and so guarantee ordering. Due the ordering it may still become a bottle-neck. If ordering is not a requirement for your use-case you may want to consider using UnorderedThreadPoolEventExecutor to maximize the parallelism of the task execution. "*

<br>

**Thread safety**

`ChannelPipeline` 은 Thread-Safe 하다.

- `ChannelHandler`는 언제든지 `ChannelPipeline` 에 더해지거나 제거될 수 있다.

> *" For example, you can insert an encryption handler when sensitive information is about to be exchanged, and remove it after the exchange. "*

<br><br>

## EventLoop

일단 등록되면, `Channel`을 위한 모든 I/O 작업을 처리한다.

- 무한 루프(?) 속에서 Event 를 수신하여 처리한다.

> " Netty는 Channel에서 발생하는 이벤트들을 EventLoop가 처리하는 구조입니다. " 
> 
> [출처 : Netty의 스레드 모델](https://effectivesquid.tistory.com/65)

하나의 `EventLoop` 객체는 일반적으로 하나 이상의 Channel 객체를 처리할 것이다.

- Implementation details, internals 에 따라 다를 수 있다.

<br>

```java
public interface EventLoop extends OrderedEventExecutor, EventLoopGroup {
    @Override
    EventLoopGroup parent();
}
```

```java
public class DefaultEventLoop extends SingleThreadEventLoop {

    public DefaultEventLoop() {
        this((EventLoopGroup) null);
    }
    
    ...

    @Override
    protected void run() {
        for (;;) {
            Runnable task = takeTask(); // taskQueue(이벤트 큐)에서 Task(Event) 를 가져온다.
            if (task != null) {
                runTask(task);
                updateLastExecutionTime();
            }

            if (confirmShutdown()) {
                break;
            }
        }
    }
}

```

```java
public abstract class SingleThreadEventLoop extends SingleThreadEventExecutor implements EventLoop {

    protected static final int DEFAULT_MAX_PENDING_TASKS = Math.max(16,
            SystemPropertyUtil.getInt("io.netty.eventLoop.maxPendingTasks", Integer.MAX_VALUE));

    private final Queue<Runnable> tailTasks;

    ...
```

```java
public abstract class SingleThreadEventExecutor extends AbstractScheduledEventExecutor implements OrderedEventExecutor {

    ...

    private final Queue<Runnable> taskQueue; // EventQueue

    ...

```

> **위 내용을 보면 각각의 EventLoop 가 EvnetQueue 를 가지고 있음을 알 수 있다.**

<br><br>

## EventLoopGroup

> " Special EventExecutorGroup which allows registering Channels that get processed for later selection during the event loop. "

이벤트 루프 동안 나중에 선택하기 위해 처리되는 채널을 등록할 수 있는 특수 EventExecutorGroup입니다.

`EventLoop` 는 `EventLoopGroup` 을 상속(확장)한다.

> 무슨 의미인지...?
> 
> '나중에 처리할 수 있는 이벤트를 등록할 수 있다'는 의미일까? 

```java
public interface EventLoopGroup extends EventExecutorGroup {
    /**
     * Return the next {@link EventLoop} to use
     */
    @Override
    EventLoop next();

    /**
     * Register a {@link Channel} with this {@link EventLoop}. The returned {@link ChannelFuture}
     * will get notified once the registration was complete.
     */
    ChannelFuture register(Channel channel);

    /**
     * Register a {@link Channel} with this {@link EventLoop} using a {@link ChannelFuture}. The passed
     * {@link ChannelFuture} will get notified once the registration was complete and also will get returned.
     */
    ChannelFuture register(ChannelPromise promise);

    /**
     * Register a {@link Channel} with this {@link EventLoop}. The passed {@link ChannelFuture}
     * will get notified once the registration was complete and also will get returned.
     *
     * @deprecated Use {@link #register(ChannelPromise)} instead.
     */
    @Deprecated
    ChannelFuture register(Channel channel, ChannelPromise promise);
}
```

<br><br>

## Note

**EventLoop, EventLoopGroup 은 무엇인가?**

EventLoop 가 EventLoopGroup 을 상속한다.

> 근데 EventLoopGroup 의 메서드를 보면 뭔가 abstract 클래스의 느낌이 들기도 하는 듯 하다.

> 흠...

<br>

**Channel <-> EventLoop 관계는 무엇인가?**

> " Netty의 이벤트는 Channel에서 발생합니다. "
> 
> " Netty의 Channel은 하나의 이벤트 루프에 등록됩니다. " 
> 
> " 하나의 이벤트 루프 스레드에는 여러 채널이 등록될 수 있습니다. "
> 
> " 각각의 이벤트 루프 객체는 개인의 이벤트 큐를 가지고 있습니다. "
> 
> " Netty는 Channel에서 발생하는 이벤트들을 EventLoop가 처리하는 구조입니다. " 
> 
> " 다중 스레드 이벤트 모델에서 이벤트의 실행 순서가 일치하지 않는 근본적인 이유는 이벤트 루프들이 이벤트 큐를 공유하기 때문에 발생하는데 Netty는 이벤트 큐를 이벤트 루프 스레드의 내부에 둠으로써 실행 순서 불일치의 원인을 제거한 것입니다. "
> 
> [출처 : Netty의 스레드 모델](https://effectivesquid.tistory.com/65)


정리하면,

1. Channel 에서 Event 가 발생한다.
   1. Channel 은 (이미)EventLoop 가 지정되어 있는 상태이다.
   2. 하나의 EventLoop 는 여러 Channel 을 처리할 수 있다.
2. EventLoop 내부의 EventQueue 에 Event 가 적재된다.
3. EventQueue 에 쌓인 Event 를 처리한다.



**Chanele은 EventLoop가 지정되어 있는 상태이다.**
```java
public interface Channel extends AttributeMap, ChannelOutboundInvoker, Comparable<Channel> {
    ChannelId id();

    EventLoop eventLoop();

    ...
}

public abstract class AbstractChannel extends DefaultAttributeMap implements Channel {

    ...

    @Override
    public EventLoop eventLoop() {
        EventLoop eventLoop = this.eventLoop;
        if (eventLoop == null) {
            throw new IllegalStateException("channel not registered to an event loop");
        }
        return eventLoop;
    }
```

> `EventLoop` 가 등록되지 않은 `Channel`은 `IllegalStateException` 이 발생한다.


<br><br>

## 참고

> " 위의 이벤트 루프 모델을 잘 살펴보면 Netty를 이용하여 개발 할 때 주의해야할 점이 한 가지 있습니다. 바로 이벤트 루프 스레드가 blocking되면 안되는 것인데요. 이벤트 루프 스레드가 blocking되어 버리면 해당 이벤트 루프에 등록된 Channel들에서 발생한 이벤트들이 제때 처리되지못하고 요청들이 밀려버리는 상황이 발생합니다.
> 
> (... 중략)
> 
> Netty에서는 이런 blocking작업을 어떻게 처리해야할까요? Netty는 이벤트 루프가 blocking되지 않게 blocking구간이 있는 ChannelHandler를 별도의 EventExecutor에서 실행될 수 있도록 지원합니다. " 
> 
> [출처 : Netty의 스레드 모델](https://effectivesquid.tistory.com/65)