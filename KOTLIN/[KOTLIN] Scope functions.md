**(객체의 컨텍스트 내에서) 임시 scope 를 생성하여 코드 블록(일련의 코드들)을 실행하기 위한 기능**

## 다음과 같이 5개의 범위 함수(scope function)가 존재한다. <br>
> [공식 문서 - scope-functions/function-selection](https://kotlinlang.org/docs/scope-functions.html#function-selection)

|Function|Object reference|Return value|Is extension function?|(TIP) 선택 기준|
|-|-|-|-|-|
|`let`|it|Lambda result|O|1. non-null 객체의 lambda 실행 시 <br> 2. local 범위의 변수로 표현식 사용 시|
|`run`|this|Lambda result|O|1. 객체 configuration + result 계산(반환)|
|`run`|-|Lambda result|X <br> called without the context object|1. 표현식 실행 (Running statements where an expression is required)|
|`with`|this|Lambda result|X <br> takes the context object as an argument|1. 객체에 대한 그루핑(연관된 여러 개의 함수?) 함수 호출 (Grouping function calls on an object)|
|`apply`|this|Context object|O|1. 객체 configuration (Object configuration)|
|`also`|it|Context object|O|1. 추가적인 효과(코드)를 부여할 때|


### 공통점

1. '하나의 객체로 코드 블록(일련의 코드들)을 실행'한다는 것 (간결성, 가독성 향상)

### 차이점 

**범위 함수들이 각자 비슷하기 때문에 차이를 알아두는 것 중요**

1. 블록 내에서 객체를 사용하는 방식 (The way to refer to the context object.)
   - `it` : as a lambda argument
   - `this` : as a lambda receiver
2. 전체 표현식의 결과 (The return value.)

### 주의점

1. 범위 함수(scope function) 의 사용 여부, 사용 기준은 각자의 회사/팀에서 결정하여 적절히 사용하면 된다.
2. '남용' 피한다.
3. '중첩 사용'은 피한다. ('체이닝' 시 주의한다.)

<br><br>

## 사용 예시를 살펴보자.

```kotlin
// 일반 코드 (without scope function)
val person = Person("name", 20, "city")
println(person)
person.incrementAge()    // 20 -> 21
person.moveTo("newCity") // city -> newCity
println(person)
```

```kotlin
// 수정 코드 (with scope function)
val person = Person("name", 20, "city).let {
    println(it)
    it.incrementAge()    // 20 -> 21
    it.moveTo("newCity") // city -> newCity
    println(it)
}
```

<br><br>

## `it` vs `this`

- 둘 다 동일한 기능을 제공한다.
- 특징(장단점)을 비교하여 사용하자.

<br>

### `this`

- [lambda receiver](https://kotlinlang.org/docs/lambdas.html#anonymous-functions)
- `run`, `with`, `apply` 와 함께 사용된다.
- `this` 키워드는 생략할 수 있다.
  - 따라서, 객체 자신에 대해 특정 작업을 수행할 필요가 있을 때 사용하는 것이 권장된다.
  - `this` 생략 시, 다른 외부 변수와 헷갈릴 수 있으니 사용에 주의하자.

```kotlin
val adam = Person("Adam").apply { 
    city = "London"
    age = 20
    // 'age = 20' 은 다음과 동일하다.
    // this.age = 20 
    // adam.age = 20
}
println(adam)
```

<br>

### `it`

- lambda argument (argument 의 이름이 지정되지 않으면 기본적으로 'it' 를 사용)
- `let`, `also` 와 함께 사용된다.
- However, when calling the object functions or properties you don't have the object available implicitly like this. Hence, having the context object as it is better when the object is mostly used as an argument in function calls. it is also better if you use multiple variables in the code block.

```kotlin
fun getRandomInt(): Int {
    return Random.nextInt(100).also {
        writeToLog("getRandomInt() generated value $it")
    }
}

val i = getRandomInt()
println(i)
```

<br><br>

## Return Value

> 아래를 기억하고, 사용/결정하자.

- `apply`, `also` : return the context object
- `let`, `run`, `with` : return the lambda result

### Context object (apply, also)

- 객체 자신을 반환한다
  - **= 체이닝 가능**

```kotlin
// 예시 : 체이닝 가능하다.

val numberList = mutableListOf<Double>()
numberList.also { println("Populating the list") }
    .apply {
        add(2.71)
        add(3.14)
        add(1.0)
    }
    .also { println("Sorting the list") }
    .sort()
```
```kotlin
// 예시 : return (& assignment) 가능하다.

fun getRandomInt(): Int {
    return Random.nextInt(100).also {
        writeToLog("getRandomInt() generated value $it")
    }
}

val i = getRandomInt() // Ex: 54
```

<br>

### Lambda result

- return value 를 사용하지 않을 수 있다. (무시 OK)

```kotlin
// 에시 

val numbers = mutableListOf("one", "two", "three")
val countEndsWithE = numbers.run { 
    add("four")
    add("five")
    count { it.endsWith("e") }
}
println("There are $countEndsWithE elements that end with e.")

// [출력]
// There are 3 elements that end with e.
```

```kotlin
// 예시 : ignore the return value

val numbers = mutableListOf("one", "two", "three")
with(numbers) {
    val firstItem = first()
    val lastItem = last()        
    println("First item: $firstItem, last item: $lastItem")
}

// [출력]
// First item: one, last item: three
```

<br><br>

## Functions

### `let`

|기능|사용 가능 여부|
|-|-|
|conext object|O : `it`|
|return value|O : lambda result|

**1. (체이닝의 결과)연산 결과에 하나 이상의 함수를 호출하기 위해 사용될 수 있다.** <br>
> *"`let` can be used to invoke one or more functions on results of call chains."*

**2. non-null 변수와 함께 코드 블록(몇몇의 코드들)을 호출하기 위해 사용될 수 있다.** <br>
> *"`let` is often used for executing a code block only with non-null values"*

**3. `it` 대신 (가독성을 위해)다른 변수명을 사용할 수 있다.**

<br>

```kotlin
/** 예시 : without let */
val numbers = mutableListOf("one", "two", "three", "four", "five")

val resultList = numbers.map { it.length }.filter { it > 3 }

println(resultList)
```

```kotlin
/** 예시 : with let */
val numbers = mutableListOf("one", "two", "three", "four", "five")

numbers.map { it.length }.filter { it > 3}.let { println(it) }
```

```kotlin
/** 예시 : non-null 체크 */
val str: String? = "Hello"   
//processNonNullString(str)  // → compilation error: str can be null

val length = str?.let { 
    println("let() called on $it")        
    processNonNullString(it)  // OK: 'it' is not null inside '?.let { }'
    it.length
}
```

<br>

### `with`

|기능|사용 가능 여부|
|-|-|
|conext object|O : `this`|
|return value|O : lambda result|
|확장함수|X|

**1. return 없이 객체에 코드 블록(호출/처리들)을 수행하기 위해 사용하는 것을 권장한다.** <br>
> *" We recommend with for calling functions on the context object without providing the lambda result. "*

```kotlin
val numbers = mutableListOf("one", "two", "three")
with(numbers) {
    println("'with' is called with argument $this")
    println("It contains $size elements")
}
```

<br>

### `run`

|기능|사용 가능 여부|
|-|-|
|conext object|O : `this`|
|return value|O : lambda result|

**1. 객체 초기화 / 결과 값(return value) 계산 시 사용하는 것을 권장한다.**

> *" `run` is useful when your lambda contains both the object initialization and the computation of the return value. "*

```kotlin
val service = MultiportService("https://example.kotlinlang.org", 80)

val result = service.run {
    port = 8080
    query(prepareRequest() + " to port $port")
}

// the same code written with let() function:
val letResult = service.let {
    it.port = 8080
    it.query(it.prepareRequest() + " to port ${it.port}")
}
```

> **확장 함수 / non-확장 함수 관련된 내용에 대해서는 다시 확인해볼 것** <br>
> https://kotlinlang.org/docs/scope-functions.html#run

<br>

### `apply`

|기능|사용 가능 여부|
|-|-|
|conext object|O : `this`|
|return value|O : context object|

**1. 결과 값(return value)을 반환하지 않고, 객체 속성에 대해 작업이 필요할 때 사용하는 것을 권장한다.** <br>
> *" Use apply for code blocks that don't return a value and mainly operate on the members of the receiver object. "*

```kotlin
val adam = Person("Adam").apply {
    age = 32
    city = "London"
}

println(adam) // Person(name=Adam, age=32, city=London)
```

<br>

### `also`

|기능|사용 가능 여부|
|-|-|
|conext object|O : `it`|
|return value|O : context object|

**1. 객체를 받아 (객체에 대한 작업이 아닌) 작업들을 수행할 때 사용하는 것 권장한다.** <br>
> *" Use `also` for actions that need a reference to the object rather than its properties and functions, or when you don't want to shadow the this reference from an outer scope. "*

```kotlin
val numbers = mutableListOf("one", "two", "three")
numbers.also { println("The list ... $it") }.add("four") // The list ... [one, two, three]

println(numbers) // [one, two, three, four]
```

<br><br>

## `takeIf`, `takeUnless`

> [요약] <br>
> 1. '객체 반환' or 'null 반환' (← Predicate)
> 2. 객체 참조 : `it` 변수 사용

범위 함수(scope function) 외 에, 표준 라이브러리 `takeIf`, `takeUnless` 가 있다. <br>

**1. 객체에 대한 값 확인/체크가 필요할 때 주로 사용된다.**

**2. nullable 이기에, 이후 로직에서 safe call 등의 널 체크 필요하다.**

**3. `let` 함수와 같이 사용하는 유스케이스가 많다.**

**4. `it` (lambda argument) 값으로 객체 참조한다.**

|반환 값|Predicate|
|-|-|
|객체 반환|`takeIf` : `true` <br> `takeUnless` : `false`|
|null 반환|`takeIf` : `false` <br> `takeUnless` : `true`|

```kotlin
val number = Random.nextInt(100)

val evenOrNull = number.takeIf { it % 2 == 0 }
val oddOrNull = number.takeUnless { it % 2 == 0 }

println("even : $evenOrNull")
println("odd : $oddOrNull")
```

```kotlin
// 예시 : safe call

val str = "Hello"
val caps = str.takeIf { it.isNotEmpty() }?.uppercase() //val caps = str.takeIf { it.isNotEmpty() }.uppercase() //compilation error

println(caps)
```

```kotlin
// 예시 : let 조합

fun displaySubstringPosition(input: String, sub: String) {
    input.indexOf(sub).takeIf { it >= 0 }?.let {
        println(input)
        println(it)
    }
}

displaySubstringPosition("010000011", "11")
displaySubstringPosition("010000011", "12")
```

```kotlin
// 예시 : 위 코드(let 조합)를 일반적으로 작성했을 경우

fun displaySubstringPosition(input: String, sub: String) {
    val index = input.indexOf(sub)
    if (index >= 0) {
        println(input)
        println(it)
    }
}

displaySubstringPosition("010000011", "11")
displaySubstringPosition("010000011", "12")
```

<br><br>

## 참고

1. https://kotlinlang.org/docs/scope-functions.html