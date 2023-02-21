## 카카오 계정 서버의 API 배포 과정

> Rolling update

![](../../images/[Backend]%20JVM%20warm%20up_33.png)

> Application Ready - Warm Up 사이에 n초 delay를 줄 수 있다.

![](../../images/[Backend]%20JVM%20warm%20up_46.png)

<br>

## Compilation Level 

![](../../images/[Backend]%20JVM%20warm%20up_51.png)

<br>

## JIT

![](../../images/[Backend]%20JVM%20warm%20up_03.png)

![](../../images/[Backend]%20JVM%20warm%20up_36.png)

위 과정에서 C1, C2는 별도 쓰레드로 동작한다.

<br>

C2 컴파일러의 큐가 가득차면, C1의 Level2 로 컴파일한다. 이후 여유가 생기면 Level3, 4 컴파일 처리가 진행된다.

Level2 컴파일이 많이 발생한다면, C2 컴파일러 큐가 가득찼다는 것을 알 수 있겠다. 이때는 C2 컴파일러 쓰레드 수를 조정할 필요가 있겠다.

![](../../images/[Backend]%20JVM%20warm%20up_13.png)

<br>

## Code Cache

![](../../images/[Backend]%20JVM%20warm%20up_21.png)

초기 캐시 사이즈와 최대 사이즈 조정이 가능하다. 

코드 캐시가 가득차면 성능 상 이점을 볼 수 없다. 아래와 같은 메시지가 발생하면 사이즈를 늘려주어야 한다.

![](../../images/[Backend]%20JVM%20warm%20up_14.png)

> 카카오 계정 서버에서는 위 메시지는 발생하지 않았다. 기본 값을 그대로 사용했다. (기본 값은 몇인지 확인해보자.)

<br>

## Compilation Level Threshold

다음의 지표들을 확인해볼 수 있다.

![](../../images/[Backend]%20JVM%20warm%20up_58.png)

> - InvocationThreshold : 메서드 호출 수
> - BackEdgeThreshold : 메서드 내 반복문 수
> - CompileThreshold : InvocationThreshold + BackEdgeThreshold (?)


<br>

## JIT Log

아래 옵션을 활성화하여 컴파일 로그를 확인할 수 있다.

![](../../images/[Backend]%20JVM%20warm%20up_04.png)

![](../../images/[Backend]%20JVM%20warm%20up_34.png)

![](../../images/[Backend]%20JVM%20warm%20up_49.png)

<br>

## 결론

카카오 계정 서버에서는 다음과 같이 지표를 확인했다. 

warm up 시간도 고려해야 하기 때문에 결론적으로 250번 진행하는 것을 선택했다. (지연 문제가 발생하지 않는 선에서, 짧은 시간 선택)

![](../../images/[Backend]%20JVM%20warm%20up_40.png)