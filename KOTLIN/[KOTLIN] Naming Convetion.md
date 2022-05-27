## [Source code organization](https://kotlinlang.org/docs/coding-conventions.html#source-code-organization)

### Source file names

UpperCamelCase

<br>

### Source file organization

Placing multiple declarations (classes, top-level functions or properties) in the same Kotlin source file is encouraged as long as these declarations are closely related to each other semantically, and the file size remains reasonable (not exceeding a few hundred lines).
In particular, when defining extension functions for a class which are relevant for all clients of this class, put them in the same file with the class itself. When defining extension functions that make sense only for a specific client, put them next to the code of that client. Avoid creating files just to hold all extensions of some class.

**요약**

- 모든 클래스에서 사용되는 '확장 함수' : 해당 클래스 내에 선언
- 일부 클래스에서 사용되는 '확장 함수' : 클라이언트 클래스 쪽에 선언
- 모든 확장함수에 대해 파일을 생성하는 것은 피할 것 

<br>

### Class layout (order)
 
1. Property declarations (Initializer blocks)
2. Secondary constructors
3. Method declarations
4. Companion Object

Do not sort the method declarations alphabetically or by visibility, and do not separate regular methods from extension methods. Instead, put related stuff together, so that someone reading the class from top to bottom can follow the logic of what's happening. Choose an order (either higher-level stuff first, or vice versa) and stick to it.

Put nested classes next to the code that uses those classes. If the classes are intended to be used externally and aren't referenced inside the class, put them in the end, after the companion object.

**요약**

1. Property (초기화 블록)
2. (Secondary) 생성자
3. Method
4. Companion 객체

- 메서드 작성 시, '알파벳', '접근제어자' 순으로 작성 X
  - '관계가 있는' 메서드 순으로 작성 (자연스럽게 로직을 이해할 수 있게)
- 해당 클래스를 사용하는 코드 옆에 중첩 클래스 배치 (??)
- 클래스가 외부에서 사용되도록 의도되고 클래스 내부에서 참조되지 않는 경우 : Companion 객체 뒤에 작성

<br>

### Interface implementation layout

When implementing an interface, keep the implementing members in the same order as members of the interface (if necessary, interspersed with additional private methods used for the implementation).

**요약**

- 인터페이스, 구현체의 작성 순서 : 동일

<br>

### Overload layout

Always put overloads next to each other in a class.

**요약**

- '오버로딩'은 붙여서 작성

<br><br>

## [Naming rules](https://kotlinlang.org/docs/coding-conventions.html#naming-rules)

### Package & Class

Names of packages are always lowercase and do not use underscores (org.example.project). Using multi-word names is generally discouraged, but if you do need to use multiple words, you can either just concatenate them together or use camel case (org.example.myProject).

Names of classes and objects start with an uppercase letter and use camel case.

**요약**

- 패키지 : lowercase
  - 복수 단어 사용 시 : (1)붙여쓰거나, (2)lowerCamelCase
  - 단, 최대한 복수 단어 사용을 피하자.
- 클래스(객체) : UpperCamelCase

<br>

### Function names

Names of functions, properties and local variables start with a lowercase letter and use camel case and no underscores.

Exception: factory functions used to create instances of classes can have the same name as the abstract return type.

**요약**

- Function & Property & Local variable : lowerCamelCase
  - 단, (객체를 생성하는)팩토리 메서드 : 그 클래스와 동일한 이름을 가질 수 있음 (??)

```kt
interface Foo { /*...*/ }

class FooImpl : Foo { /*...*/ }

fun Foo(): Foo { return FooImpl() }
```

<br>

### Names for test methods

In tests (and only in tests), you can use method names with spaces enclosed in backticks. Note that such method names are currently not supported by the Android runtime. Underscores in method names are also allowed in test code.

**요약**

- (\` 으로 감싼) 공백 가능
- underscore 허용

<br>

### Property names

Names of constants (properties marked with const, or top-level or object val properties with no custom get function that hold deeply immutable data) should use uppercase underscore-separated (screaming snake case) names.

```kt
const val MAX_COUNT = 8
val USER_NAME_FIELD = "UserName"
```

Names of top-level or object properties which hold objects with behavior or mutable data should use camel case names:

```kt
val mutableCollection: MutableSet<String> = HashSet()
```

Names of properties holding references to singleton objects can use the same naming style as object declarations.

```kt
val PersonComparator: Comparator<Person> = /*...*/
```

For enum constants, it's OK to use either uppercase underscore-separated names (screaming snake case) (enum class Color { RED, GREEN }) or upper camel case names, depending on the usage.

**요약**

- 상수(const + val, val) : UPPER_CASE (underscore-separated)
- Mutable 객체, 속성 : lowerCamelCase (??)
- Singleton 객체 : UperCamelCase (??)
- ENUM 
  - UPPER CASE(underscore-separated)
  - UPPER CAMEL CASE

<br>

### Names for backing properties

If a class has two properties which are conceptually the same but one is part of a public API and another is an implementation detail, use an underscore as the prefix for the name of the private property.

```kt
class C {
    private val _elementList = mutableListOf<Element>()

    val elementList: List<Element>
         get() = _elementList
}
```

**요약**
- (하나의 이름으로) public API, 속성 사용 : (private) 속성 이름 앞에 _(underscore) 사용 (prefix)


<br><br>

> [그 외 (Formatting, Documentation comments, Avoid redundant constructs, Idiomatic use of language features, ...)](https://kotlinlang.org/docs/coding-conventions.html#formatting)
