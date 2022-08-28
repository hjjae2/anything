# [Mock Roles, not Objects](http://jmock.org/oopsla2004.pdf)

## 4. MOCK OBJECTS IN PRACTICE

시스템 설계가 약할 때(강한 결합, 잘못된 책임 등_), Mock 기반의 테스트 코드는 굉장히 복잡해지고, 많은 문제(problem)를 야기한다.

> 예를 들어, 우리가 하나의 클래스에 너무 많은 역할을 부여하고 테스트 코드(with Mocking) 작성할 때 이상함을 느끼곤한다. (그러곤 이 부분에 대해 리팩토링을 하곤 한다.) 
> 
> 이런 케이스도 위에서 말한 케이스에 포함되는 것 같다.

한 가지 대응 방법은 'Mocking 사용'을 중단하는 것이다. 

'Mocking 사용'은 설계 개선(위에서 말한 '예시' 처럼)을 위한 목적으로 사용하는 것이 더 낫다고 생각한다.

<br><br>

### 4.1 Only Mock Types You Own

`Mocking Objects` 는 설계 기법이므로 프로그래머들은 오직 그들이(프로그래머) 변경할 수 있는 타입들에 대해서만 mock을 사용해야 한다. (?)

> *" Mock Objects is a design technique so programmers should only write mocks for types that they can change. "*

> *" Programmers should not write mocks for fixed types, such as those defined by the runtime or external libraries. "* (?, 어떤 것을 의미하는 지 이해하지 못했다.)


> *" Instead they should write thin wrappers to implement the application abstractions in terms of the underlying infrastructure. Those wrappers will have been defined as part of a need-driven test. "*

<br><br>

### 4.2 Don't use getters

> *" Getters expose implementation, which increases coupling between objects and allows responsibilities to be left in the wrong module. "*

> *" Avoiding getters forces an emphasis on object behaviour, rather than state, which is one of the characteristics of ResponsibilityDriven Design. "*

Getter 는 구현체를 노출하는데, 이 구현체를 노출함으로써 결합(coupling)을 증가시키고, 책임(responsibilties)이 적절하지 않은 곳(모듈, 클래스)에 분배되게 한다.

Getter 를 피하면 객체의 상태(state)보다 동작(behaviour)을 강조하게 되는데, 이것은 `ResponsibilityDriven Design` 특징 중 하나이다.

> 내가 이해한 것은, "무분별하게 getter 를 사용하지 말자." 이다. 
> 
> 예를 들면 아래와 같을 것 같다. <br>
> A 라는 클래스(객체)에 <br>
> 1. getter 를 사용하지 않으면, A 에서 처리할 수 있다. <br>
> 2. getter 를 사용하면, getter 를 사용하는 곳에 책임(로직)이 분배된다. (= 결합을 증가시킨다.)

<br><br>

### 4.3 Be explicit about things that should not happen

> 전달하고자 한 의도를 파악한 것인지 모르겠다. (잘못된 해석일 수 있다.)

> *" There are some conditions that are not made clear when they are simply left out of the test. "*
> 
> *" A specification that a method should not be called, is not the same as a specification that doesn’t mention the method at all. "*
> 
> *" In the latter case, it’s not clear to other readers whether a call to the method is an error. We often write tests that specify that methods should not be called, even where not necessary, just to make our intentions clear. "*

'should not happen' 케이스에 대해 명시해라. 

'should not happen' 케이스는 읽는 사람에게 우리의 의도를 명확하게 밝힐 수 있다. 

의도를 분명히 밝히기 위해, 우리는 필요하지 않은 경우에도 'should not happen' 테스트를 작성한다.

<br><br>

### 4.4 Specify as little as possible in a test

> 전달하고자 한 의도를 파악한 것인지 모르겠다. (잘못된 해석일 수 있다.)

> *" When testing with Mock Objects it is important to find the right balance between an accurate specification of a unit's required behaviour and a flexible test that allows easy evolution of the code base.*
> 
> *One of the risks with TDD is that tests become “brittle”, that is they fail when a programmer makes unrelated changes to the application code.*
> 
> *They have been over-specified to check features that are an artefact of the implementation, not an expression of some requirement in the object. A test suite that contains a lot of brittle tests will slow down development and inhibit refactoring.*
> 
> *The solution is to re-examine the code and see if either the specification should be weakened, or the object structure is wrong and should be changed.*
> 
> *Following Einstein, a specification should be as precise as possible, but not more precise. "*

하나의 테스트는 가능한 작게 유지해라. (= 작은 범위만을 다뤄라.)

하나의 명세(나는 구현/메소드/테스트로 이해했다.)가 많은 것(= 기능, 확인하려는 테스트 범위 등)을 다루면 점점 다루기 힘들어진다.

예를 들어, 연관이 없다고 생각한 코드 혹은 논리적으로/기능적으로 관계가 먼 코드를 변경했을 때에도 영향을 받아 테스트가 실패할 수 있다.

복잡한 테스트 코드는 결국 개발 생산성을 저하시킨다.

이 경우 코드, 설계를 다시 확인하여 리팩토링해야 한다. (예를 들어, 강한 결합이나 낮은 응집도, 설계 등)

> " Following Einstein, a specification should be as precise as possible, but not more precise. "

> (작성 중)
