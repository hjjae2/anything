### Reflection

로드된 클래스의 정보를 찾을 수 있게 지원합니다.

구체적인 클래스(구현 클래스, 구현체)의 타입을 몰라도 그 클래스의 정보(변수, 타입, Method)에 접근할 수 있게 해주는 API입니다.

```java
//참조 클래스  구체적인 클래스
Object obj = new Car();
```

<br>

### 동작 원리

동작원리는 다음과 같습니다.

1. **JVM이 실행되면, 클래스로더에 의해 사용자가 작성한 클래스가 Method area(Class area, Static area)에 저장됩니다.**
2. Reflection API 는 이 정보를 활용합니다.

<br>

### 예시
아래는 Reflection API 를 사용한 예제입니다.

```java
// Reflection API 예제
Object obj = new Car();

Class carClass = Car.class;
Method move = carClass.getMethod("move"); // Returns a Method object that reflects the specified public member method of the class or interface represented by this Class object.

move.invoke(obj, null);
```

```java
// Reflection API 예제
Class carClass = Class.forName("Car"); // Returns the Class object associated with the class or interface with the given string name.
```

<br>

### 특징

Reflection 은 실제 운영에서 사용 시 남용하여 사용하는 것은 지양하는 것이 좋습니다. (\* 사용 시 주의가 필요합니다.)

프레임워크나 라이브러리에서 많이 사용된다고 합니다. (어떤 클래스에 사용될 지 예측할 수 없기에 동적으로 해결해주어야 한다.)

아래와 같은 곳에서 사용되고 있다고 합니다.

- Intellij 자동완성
- jackson 라이브러리
- SprinFramework's BeanFactory
- Spring Data Jpa (e.g. Entity의 `@NoArgsContsructor`)

<br>

**단점**

> 사용자가 잘못 사용했다는 전제 하에 아래와 같은 단점이 있을 수 있습니다. 
> Reflection 자체가 나쁜 것이 아니라는 것에 주의해야 합니다.

- 성능 오버헤드가 발생할 수 있습니다.
  - 이유 : 런타임에 동적으로 클래스를 분석하고 정보를 가져오기 때문에
- 추상화가 깨질 수 있습니다.
  - 이유 : private 변수, 메서드에 접근할 수 있게 되기 때문에

<br><br>

### 참고

1. [docs.oracle.com : Class `Class<T>`](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html)