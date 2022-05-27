> [Blocking-NonBlocking-Synchronous-Asynchronous](https://homoefficio.github.io/2017/02/19/Blocking-NonBlocking-Synchronous-Asynchronous/) 글을 참고합니다.

<br>

## 관심사의 차이 

|키워드|관심사|
|-|-|
|Blocking / Non-Blocking|함수가 바로 응답하는지, 안하는지 <br><br>`Blocking` : 함수 호출 후 함수가 완료될 때까지 기다린다. <br>`Non-Blocking` : 함수 호출 후 함수가 완료되는 것을 기다리지 않는다.<br>&nbsp;&nbsp;= 즉, 함수를 호출한 곳에 제어권을 곧바로 넘겨준다. <br>&nbsp;&nbsp;= 즉, 함수를 호출한 곳은 다른 일을 할 수 있다.|
|Sync / Async|함수의 '완료'를 누가 신경쓰는지 <br><br>`Sync` : 함수의 완료(응답)을 호출한 쪽에서 신경쓴다. <br>`Async` : 함수의 완료(응답)을 호출당한 쪽에서 신경쓴다. (callback 함수)|


<br><br>

## 상황별 예시

아래와 같은 컴포넌트가 있다고 가정한다.

- A : 함수 호출자
- B : 함수 피호출자

<br>

### Blocking Sync

> 1. 함수 호출 후, 응답(처리)를 기다린다. (Blocking)
> 2. 함수 호출 후, 응답을 반환받는다. (Sync)

```
A ---> B 
   |
   |  B에서 처리가 완료될 때까지 기다린다.
   |  B에서 처리가 완료되면 결과를 반환받는다.
   |
A <--- B  
```

<br>
아래와 같은 예시가 있다고 한다.

- file.read()
- file.write()
- ...

<br>

### Blocking Async

> 1. 함수 호출 후, 응답(처리)를 기다린다. (Blocking)
> 2. 함수 호출 후, 응답은 신경쓰지 않는다. 
> &nbsp;&nbsp;&nbsp;&nbsp;- 피호출자에서 callback 함수를 통해 처리한다.

```
A ---> B 
   |
   |  B에서 처리가 완료될 때까지 기다린다.
   |  B에서 처리가 완료되면 callback 함수를 통해 처리한다.
   |
A <--- B (callback)
```

이 경우는 이점이 없다. 

Blocking 된 상태이기 때문에, 굳이 callback 함수를 통해 결과를 처리받지 않아도 된다. (직접 결과를 받으면 된다.)

참고한 글에서는 Non-Blocking + Async 의 방식을 시도했지만 Blocking 처리되는 부분이 있을 때 발생할 수 있는 조합이라고 설명한다. (즉, 의도하지 않은 사용으로 인해 발생할 수 있는 조합)

> *" NonBlocking-Async 방식을 쓰는데 그 과정 중에 하나라도 Blocking으로 동작하는 놈이 포함되어 있다면 의도하지 않게 Blocking-Async로 동작할 수 있다. "*

<br>
아래와 같은 예시가 있다고 한다.

- Node.js + Mysql
<br>

### Non-Blocking Sync

> 1. 함수 호출 후, 응답(처리)를 기다리지 않는다. (Non-Blocking)
> &nbsp;&nbsp;&nbsp;&nbsp;- 함수 호출자는 다른 일을 할 수 있다.
> 2. 함수 호출 후, 응답을 반환받는다. (Sync)


```
// B에서 처리가 완료될 때까지 기다리지 않는다.
// B에서 처리가 완료되었는지 확인한다. (주기적으로 polling 하는 느낌과 비슷하다.)

A ---> B 
A --- 다른 작업
A ---> B 결과 확인
A --- 다른 작업
A ---> B 결과 확인
A --- 다른 작업
A ---> B 결과 확인
A <--- B  
```
<br>
아래와 같은 예시가 있다고 한다.

- Future.isDone() (`while(!Future.isDone()){...}`)
- ...

<br>

### Non-Blocking Asnyc

> 1. 함수 호출 후, 응답(처리)를 기다리지 않는다. (Non-Blocking)
> &nbsp;&nbsp;&nbsp;&nbsp;- 함수 호출자는 다른 일을 할 수 있다.
> 2. 함수 호출 후, 응답을 반환받는다. (Sync)


```
// B에서 처리가 완료될 때까지 기다리지 않는다.
// B에서 처리가 완료되면 callback 함수를 통해 처리한다.

A ---> B 
A --- 다른 작업
A --- 다른 작업
A --- 다른 작업
A <--- B (callback)
```
<br>
아래와 같은 예시가 있다고 한다.

- asyncFileChannel.read()
- asyncFileChannel.write()
- ...