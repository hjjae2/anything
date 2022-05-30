## Future

(미래에) 결국에는 반환받을 결과를 표현하기 위해 사용되는 인터페이스이다.

> *" The Future interface is an interface that represents a result that will eventually be returned in the future. "*

```java
package java.util.concurrent;

public interface Future<V> {
    boolean cancel(boolean var1);

    boolean isCancelled();

    boolean isDone();

    V get() throws InterruptedException, ExecutionException;

    V get(long var1, TimeUnit var3) throws InterruptedException, ExecutionException, TimeoutException;
}
```

|메서드|설명|
|-|-|
|`get()`|결과를 가져온다. <br> 결과가 아직 반환되지 않았다면, 기다린다. <br> 결과를 받기 전에 `get()` 호출하여 기다리게 된다면, 결과를 받을 때까지 'block' 한다. <br> 결과를 기다리는 것에 시간 제한을 적용시킬 수 있다.|
|`cancel()`|현재 task 를 중단한다. <br> 이미 task 가 완료되었다면, 이 명령어는 실패한다.|
|`isDone()`|task 의 현재 상태(done)를 확인한다.|
|`isCancelled()`|task 의 현재 상태(cancel)를 확인한다.|

<br>

### 구현체

- CompletableFuture
- ForkJoinTask
- ...

<br><br>

### 예시 코드

```java
public static void main(String[] args) throws InterruptedException, ExecutionException {
    ExecutorService executorService = Executors.newFixedThreadPool(2);

    Future<String> dataReadFuture = executorService.submit(() -> {
        System.out.println("Reading data...");
        TimeUnit.SECONDS.sleep(5);
        return "Data reading finished";
    });
    Future<String> dataProcessFuture = executorService.submit(() -> {
        System.out.println("Processing data...");
        TimeUnit.SECONDS.sleep(5);
        return "Data is processed";
    });

    while (!dataReadFuture.isDone() && !dataProcessFuture.isDone()) {
        System.out.println("Reading and processing not yet finished.");
        // Do some other things that don't depend on these two processes
        // Simulating another task
        TimeUnit.SECONDS.sleep(1);
    }

    System.out.println(dataReadFuture.get());
    System.out.println(dataProcessFuture.get());
}

// [결과 출력 : 5sec 39ms]
// Reading data...
// Processing data...
// Reading and processing not yet finished.
// Reading and processing not yet finished.
// Reading and processing not yet finished.
// Reading and processing not yet finished.
// Reading and processing not yet finished.
// Data reading finished
// Data is processed
```

<br>

### 중요 포인트 : isDone()

`while(isDone())` 구문은 사용해도 되고, 사용하지 않아도 된다. 

하지만 위의 예시에서 `while(isDone())` 을 사용하지 않고, 곧바로 `get()`을 호출하면 대략 5초 간의 Blocking 상태를 맞이하게 될 것이다. (= 자원의 낭비가 될 수 있다.)

> *" What's important here is the usage of the isDone() method. If we didn't have the check, there wouldn't be any guarantee that the results were packed in the Futures before we've accessed them. If they weren't, the get() methods would block the application until they did have results. "*

<br><br>

## CompletableFuture

**목적** <br>
**1. 비동기 처리 후 (`Future`)** <br>
**2. 다음 로직 실행 + Error 핸들링 (`CompletionStage`)**

```java
public class CompletableFuture<T> implements Future<T>, CompletionStage<T> { 
    ...
}
```

### 예시 코드

```java
CompletableFuture.supplyAsync(() -> {
    // Supplier::get()
    log.info("Supplier::get()");

    return "First";
}).thenApplyAsync(s -> {
    // Function::apply()
    log.info("Function::apply() " + s);

    return s + " " + "Second";
}).thenAcceptAsync(s -> {
    // Consumer::accept()
    log.info("Consumer::accept() " + s);
}).exceptionally(e -> {
    // Function<Throwable, ? extends T>::apply()
    e.printStackTrace();
    return null;
});


// [결과 출력]
// [ForkJoinPool.commonPool-worker-3] INFO com.example.demo.future.FutureTest - Supplier::get()
// [ForkJoinPool.commonPool-worker-5] INFO com.example.demo.future.FutureTest - Function::apply() First
// [ForkJoinPool.commonPool-worker-5] INFO com.example.demo.future.FutureTest - Consumer::accept() First Second
```

<br><br>

## 참고

1. https://stackabuse.com/guide-to-the-future-interface-in-java/
2. https://umanking.github.io/2020/10/15/java-completable-future/