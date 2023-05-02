## 객체를 수정하기 위해서, (메서드에서) 객체를 파라미터로 받아 메서드 내부에서 해당 객체를 수정하지 말자.


어떤 클래스를 파라미터로 받으면 결합도가 높아진다.


다음은 (Car 객체를 필드로 갖는) CarContext 객체의 Car 필드를 새로운 값으로 설정하는 예시이다.

> "  아래와 같이, 함수 내부에서 만드는 새로운 값 또는 상태를 함수의 안에서 외부 객체의 상태 변경에 직접 적용하는 것은 좋지 않습니다. 가능하면 리턴으로 받아서 처리하는 방식이 좋습니다. "

```java
// Bad
Result createCar(CarContext carContext, Something A, Something B) {
  ...
  carContext.setCar(new Car());
  ...
  return new Result(someValue);
}
```

```java
// Good
Pair<Result, Car> createCar(Something A, Something B) {
  ...
  return new Pair<>(new Result(someValue), new Car());
}
```

위 코드는 (새롭게 설정할)Car 객체를 받아서, 메서드 외부에서 설정한다.

**정리하면, 객체(클래스)에 어떤 새로운 값을 설정할 때 메서드의 인자로 넘겨, 메서드 내부에서 처리하지 말자. 설정할 새로운 값을 받아와서 처리하자. :star::star:**

<br>

## (가능하면) 메서드에서 클래스(객체)를 받지 말고, 필요한 것만 명시적으로 받자.

메서드에 어떤 객체를 넘겼을 때, 메서드를 사용하는 사용처에서는 메서드 내에서 해당 객체에 어떤 행위(값을 수정한다거나, 어떤 값에 접근한다거나 등등)를 하는지 알 수가 없다.

즉, 객체에 어떤 행위를 하는지 알 수 없어 불안하다.

> 물론 메서드명을 직관적으로 지어, 추측할 수 있을 것이긴 하다.

이를 보완하기 위해서는 메서드에서 필요한 값만 받아 사용하는 것이 좋다.

```java
// Bad
void doSomethingForPassengers(CarContext carContext, Something A, Something B) {
  List<People> passengers = carContext.getPassengers();
  SomeValue someValue = carContext.getSomeValue();
  …
}
```

```java
// Good
void doSomethingForPassengers(List<People> passengers, SomeValue someValue, Something A, Something B) {
  ...
}
```

정리하면 다음과 같은 이점이 있다.

1. 메서드 사용처에서 객체를 넘겼을 때의 불안감을 제거할 수 있다.
2. 객체를 넘겼을 때 보다 더 직관적(명시적)이다. (메서드의 의도가 명확해진다.) 
3. **코드 간 결합도가 줄어든다.**


**이때 파라미터의 수가 계속 증가해서 불편할 수 있다. 이런 경우 일단은 변경하고(외부 종속을 끊고) 함수를 더 작은 책임 단위로 리팩터링 하는 것을 검토해볼 수 있다. :thumbsup:**

> 꿀팁이다. :thumbsup:

<br>

## 루프 최적화를 위해서 캐싱하고 싶다면 Context에 넣지 말고, 루프 밖으로 뺄 수 있는지부터 보자

> 요약 : (캐시를 위해)전역 변수와 같은 형태로 사용하지 말자

<br>

여기까지 위 내용의 핵심을 정리해보면 **"메서드에 객체를 넘기지 말자"** 인 것 같다.

---

## 고차 함수로 의존성 줄이기

> 서비스(클래스) 간 의존성을 고차 함수를 통해 제거할 수 있다. :star::star:
>   
> 다만, 이번 글은 서비스 간에 순환 의존성이 있을 때 이것을 제거하기 위한 방법으로 소개됐다. 

```java
// Before
@Service
public class ServiceA {
    @Resource
    ServiceB serviceB;

    public void methodA(Integer paramFirst) {
        Output.printf("'pass %d to ServiceB and get %s' by ServiceA\n", paramFirst, serviceB.methodB(paramFirst));
    }

    public Integer getValue() {
        return 10;
    }
}
```

```java
// After
@Service
public class ServiceA {
    public void methodA(Integer paramFirst, Function<Integer, Integer> methodB) {
        Output.printf("'pass %d to ServiceB and get %s' by ServiceA\n", paramFirst, methodB.apply(paramFirst));
    }

    public Integer getValue() {
        return 10;
    }
}
```

> *" ServiceA::methodA()의 시그니처를 수정하고, serviceB.methodB() 메서드 호출을 함수형 인터페이스 apply() 호출로 변경합니다. "*

위의 예시 서비스(ServiceA)를 사용하는 사용처에서는 다음과 같이 변경될 것이다.

```java
// Before
@Component
public class Handler {
    private final ServiceA serviceA;

    public Handler(final ServiceA serviceA) {
        this.serviceA = serviceA;
    }

    public void execute(long count) {
        for (long cnt = 0; cnt < count; cnt++) {
            serviceA.methodA(2);
        }
    }
}
```

```java
// After
@Component
public class Handler {
    private final ServiceA serviceA;
    private final ServiceB serviceB;

    public Handler(final ServiceA serviceA, final ServiceB serviceB) {
        this.serviceA = serviceA;
        this.serviceB = serviceB;
    }

    public void execute(long count) {
        for (long cnt = 0; cnt < count; cnt++) {
            serviceA.methodA(2, serviceB::methodB);
        }
    }
}
```

추가로 위와 같이 리팩터링하면 단위 테스트를 작성할 때도 의존성이 제거될 것이다. (조금 더 작성이 쉬워질 것이다.)

> **Service A 에서의 의존성은 제거됐지만, 해당 의존성들은 Handler에 추가됐다.**
> 
> **이 글의 목적이 '순환 의존성을 제거하기 위한 방법'인 부분을 기억하자.**

<br>

**위 리팩터링에서 주의할 점이 있다.**

메서드를 넘기면서 성능 차이가 발생할 수 있다.

자세한 내용은 https://tech.kakao.com/2023/01/19/kakaotalk-java-app-server-refactoring/ 글을 참고한다.


---

## 코드 복잡도 줄이기 (Cyclomatic Complexity, NPath Complexity)

> 위의 내용 (메서드에 객체를 넘기지 않는 것, 고차 함수로 의존성을 줄이는 것)을 적용하면, 이미 코드 내 의존성들이 많이 정리된 상태가 된다. 
> 
> (코드가 정리된)이후 본격적으로 코드의 복잡도를 줄여봤다는 내용이다.

코드 복잡도를 수치로 계산하는 다양한 방법들이 있지만, 그 중 Cyclomatic Complexity(이하 CC), NPath Complexity(이하 NPath)를 기준으로 정리했다.

> *" CC는 함수에 제어문(분기, 루프 등)이 없다면 1점, 있다면 제어문마다 1점을 부여합니다. 또한, 조건식 안의 논리식도 1점으로 계산하여 각각의 점수를 모두 더합니다. NPath는 코드를 실행할 수 있는 비순환 경로의 수를 의미합니다. "*

> *" 코드의 복잡도를 줄이는 건 반드시 특별할 필요가 없습니다. **기본은 동일합니다. 하나의 함수에 많은 코드가 있다는 건, 코드 내부에서 필요 이상의 책임을 가지고 있다는 것입니다. 함수가 현재 가지고 있는 많은 책임을 더 작은 단위의 책임으로 나눈 뒤, 각 함수가 각 1개의 책임만 담당하도록 한다면, 복잡도 역시 각 함수가 나누어 가지게 됩니다. 그러면 이후에 추가되는 개별 함수들의 복잡도 또한 낮아지게 됩니다.**  "* :star::star:

<br>

### 함수 추출하기

```java
public Data buildData(boolean isConditionA, boolean isConditionB, boolean isConditionC, String extraCondition) {
    int someValue;
    if (isConditionA) {
        someValue = 10;
    }
    else {
        if (extraCondition.equals("ForceB") || isConditionB) {
            someValue = 20;
        } else {
            if (isConditionC) {
                someValue = 30;
            } else {
                someValue = 40;
            }
        }
    }

    Data data = new Data();
    data.setA(someValue + 1);
    if (someValue == 30) {
        data.setB(someValue + 2);
    } else {
        data.setB(someValue + 4);
    }
    data.setC(someValue + 3);
    return data;
}
```

위 메서드(buildData)는 2가지 책임을 갖고 있다.

1. someValue 를 구하는 것
2. Data 객체를 생성하는 것

이것을 각각의 메서드로 추출할 수 있다.

1. someValue 를 구하는 것 : `getSomeValue()`
2. Data 객체를 생성하는 것 : `makeData()`

> 잘 해가고 있는 것 같다. 
> 
> 다만 간혹 추출된 메서드가 많아 해당 클래스를 읽을 때 혼잡할 것 같다는 생각이 들 때가 있다. 이때는 클래스의 책임이 큰 것일 수 있겠다.

<br>

### 중첩 조건문을 보호 구문으로 바꾸기

```java
// Bad
private int getSomeValue(boolean isConditionA, boolean isConditionB, boolean isConditionC, String extraCondition) {
    if (isConditionA) {
        return 10;
    }
    else {
        if (extraCondition.equals("ForceB") || isConditionB) {
            return 20;
        } else {
            if (isConditionC) {
                return 30;
            } else {
                return 40;
            }
        }
    }
}
```

```java
// Better
private int getSomeValue(boolean isConditionA, boolean isConditionB, boolean isConditionC, String extraCondition) {
    if (isConditionA) {
        return 10;
    }
    if (extraCondition.equals("ForceB") || isConditionB) {
        return 20;
    }
    if (isConditionC) {
        return 30;
    }
    return 40;
}
```
> *" 코드가 한결 보기 편해졌습니다. 혹 Complexity reducer Plugin을 활성화해 두고 리팩토링 작업을 따라왔다면 중첩된 if 문들을 정리하면서 getSomeValue() 함수의 NPath가 4에서 8로 2배가 증가한 것을 확인할 수 있었을 것입니다. 코드의 로직은 동일하고 가독성도 좋아졌는데 오히려 코드 복잡도를 나타내는 수치가 증가했습니다. 그 이유는 첫 번째 if 문의 조건이 만족하면 바로 return 하여 함수를 나오기 때문에, 두 번째 if 문을 타지 않는다는 것을 계산에 포함하지 않고 나온 수치라서 그렇습니다. 계산 수치를 개선하기 위해 다시금 아래와 같이 수정해 보겠습니다. "*

```java
// Best
private int getSomeValue(boolean isConditionA, boolean isConditionB, boolean isConditionC, String extraCondition) {
    if (isConditionA) {
        return 10;
    }
    else if (extraCondition.equals("ForceB") || isConditionB) {
        return 20;
    }
    else if (isConditionC) {
        return 30;
    }
    return 40;
}
```

if-else 구문으로 변경을 통해 if 문의 조건이 만족될 경우 다음 if가 타지 않음을 명시할 수 있다. (복잡도 수치도 개선된다.)