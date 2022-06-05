### 코틀린에서는 nullable, non-nullable 타입을 구분하여 사용한다.

**non-nullable**

```kotlin
var a: String = "abc"
a = null // X : 불가능 (컴파일 오류 발생)

// 이 경우 변수 a 에는 non-null 이 보장되기에 아래와 같은 문법 사용 가능하다.
println(a.length) // O : 가능
```

<br>

**nullable**

```kotlin
var a: String? = "abc"
a = null // O : 가능

// 이 경우 변수 a 에는 non-null 이 보장되지 않기에 아래와 같은 문법 사용 불가능하다.
println(a.length) // X : 불가능 (컴파일 오류 발생)
```

<br>

### 아래(nullable) 예시의 경우, property 를 접근/사용하기 위한 3가지 방법이 있다.

**1. 명시적 null 체크**

```kotlin
println(if (a != null) a.length else 0) 

// 출력 : 0
```

**참고: Elvis Operator**

if else 구문은 elvis operator 로 대체할 수 있다.

```kotlin
// println(if (a != null) a.length else 0) 

println(a?.length ?: 0)
```

<br>

**2. Safe calls(`?.`)**

```kotlin
// a 가 
// null : null
// non-null : a.length
println(a?.length)

// 출력 : null
```

<br>

safe call 의 경우 아래와 같이 chain 형태로 유용하게 사용할 수 있다.

```kotlin
person?.department?.head?.name
```

<br>

"safe call(non-null 보장) + 로직 수행" 을 위해, `let` 을 사용할 수 있다.

```kotlin
a?.let { println("hi") }
```

<br>

safe call 의 경우, assignment 위치에서 사용하는 것도 가능하다.

```kotlin
person?.department?.head = managersPool.getManager() 

// person, department 중 하나가 null 인 경우 getManager() 은 실행되지 않는다.
```

> null 이 포함되면 assignment 는 생략된다.
> 
> 표현식(함수)의 경우, 실행되지 않음에 주의한다.

<br>

**3. Not-null assertion operator `!!`**

- non-null type 컨버팅
- 단, null 일 경우 NPE 발생

```kotlin
// a!! 는 String? 을 String 으로 컨버팅한다.
// 단, a 가 null 인 경우 NPE 발생한다.
println(a!!.length)
```

<br><br>

### [기타] Safe casts

Safe cast 통해 ClassCastException 대신 null 을 반환할 수 있다.

타입 캐스팅 시 잘못된 타입일 경우 → `ClassCastException` 발생

> null 인 경우, NPE 발생 
> 
> 예시 : "null cannot be cast to non-null type kotlin.Int"

```kotlin
// 잘못된 타입인 경우, null 반환
val aInt: Int? = a as? Int
```

<br>

### [기타] Collections of a nullable type

nullable 요소를 가진 컬렉션의 경우, null 과 관련된 처리를 할 수 있는 여러 함수가 있다.

```kotlin
val nullableList: List<Int?> = listOf(1, 2, null, 4)
val intList: List<Int> = nullableList.filterNotNull()
```

<br><br>

### 참고

1. https://kotlinlang.org/docs/null-safety.html#safe-casts