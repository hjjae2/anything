## JVM

자바 가상 머신(Java Virtual Machine)이다. (스택 기반의 가상 머신이다.)

> \* 여기서 가상 머신이란? <br> 프로그램을 실행하기 위해 물리적인 기계(머신)와 유사한 (논리적인? 가상의?) 머신을 소프트웨어로 구현한 것이다.

Java Application 을 클래스 로더를 통해 읽어 들이고, Java API 와 함께 실행한다.

Java 와 OS 사이에서 중개 역할을 한다. Java 가 OS 에 구애받지 않고 사용할 수 있는 환경을 제공한다.

이 외 메모리 관리, GC 을 수행한다.

<br>

### 자바 프로그램의 실행 과정

1. 프로그램이 실행되면 JVM 이 OS 로부터 메모리를 할당 받는다. 용도에 따라 메모리 영역을 나눈다.

2. javac(자바 컴파일러)가 자바 소스코드(.java)를 자바 바이트코드(.class)로 변환한다.

3. 클래스 로더(Class Loader) 를 통해 바이트코드(.class) 를 로딩한다.

4. 로딩된 바이트 코드(파일)들은 Execution (엔진) 을 통해 해석된다.

5. 해석된 바이트 코드(파일)들은 Runtime Data Area 영역에 배치되어 실행된다.

6. 실행 중간중간 JVM이 GC, Thread Sync 와 같은 작업을 수행한다.

<br>

### JVM 구성

**클래스 로더 (Class Loader)**

JVM 내로 바이트코드(파일, .class)을 로딩한다.

Runtime 시에 동적으로 클래스를 로드한다.

jar 파일 내에 저장된 클래스들을 jvm 위에 올리고 사용하지 않는 클래스들은 메모리에서 삭제한다.

Java 는 클래스를 처음으로 참조할 때, 해당 클래스를 로드(링크)한다. 즉 런타임 시에 참조한다. (이 역할을 클래스 로더가 한다.)

<br>

**실행 엔진 (Execution Engine)**

클래스를 실행시킨다.

클래스 로더가 JVM 내의 Runtime Data Area 에 클래스(.class)를 배치시키는데, 이것들은 Execution Engine 에 의해 실행된다.

클래스 로더가 배치한 .class(자바 바이트코드)는 비교적 인간 지향적인 언어이다. 즉 완벽한 기계어가 아니라고 한다. Execution Engine 은 이 바이트 코드를 기계가 실행할 수 있도록 변환해준다. 이때 사용될 수 있는 방법은 **1. Interpreter, 2. JIT(Just-In-Time)** 이다.

1. Interpreter (인터프리터)<br>
   바이트 코드를 명령어 단위로 읽어서 실행한다. 말 그대로 인터프리터이다. 한 줄씩 수행하기 때문에 느릴 수 있다.
2. JIT (Just-In-Time)<br>
   인터프리터 방식의 단점을 보완하기 위해 도입된 JIT 컴파일러이다. 인터프리터 방식으로 실행하다가 적절한 시점에 바이트코드 전체를 컴파일하여 native 코드로 변환한다. 이후에는 더 이상 인터프리터를 사용하지 않고 native 코드로 직접 실행한다. <br><br>
   native 코드는 캐싱되기 때문에 한 번 컴파일된 코드는 빠르게 수행하게 된다. JIT 컴파일러가 전체 바이트 코드를 컴파일 하는 것은 시간이 오래걸린다. 한 번만 실행되는 코드라면 컴파일하지 않고 인터프리팅 하는 것이 유리하다. 따라서 JIT 컴파일러를 사용하는 JVM은 내부적으로 해당 메서드가 얼마나 자주 수행되는지 체크하고 일정한 수준을 넘을 때에만 JIT 컴파일을 수행한다.

<br>

**GC**

GC를 수행하는 모듈(Thread)이 있다.

**Runtime Data Area**

프로그램을 수행하기 위해 OS 로부터 할당받은 메모리 공간이다.

<br>

아래는 SpringBoot 프로젝트(gg-pigs) 의 build jar 를 해제했을 때 생기 폴더/파일이다.

<img width="466" alt="image" src="https://user-images.githubusercontent.com/35790290/121664182-2c532c00-cae2-11eb-9e0e-9cce71513338.png">

**BOOT-INF** 
- BOOT-INF/classes : 작성한 코드들이 바이트코드(.class) 형태로 존재한다.
- BOOT-INF/lib : 외부 라이브러리(jar)들이 존재한다.

**META-INF**
- META-INF/MANIFEST.MF : Start-Class, Spring-Boot-Classes, Spring-Boot-Lib, Spring-Boot-Version, Main-Class 등의 정보가 기술되어 있다.

**org.springframework.boot.loader**
- Launcher, Loader, Runner 등이 존재한다.

<br>

> Reference
> 1. https://asfirstalways.tistory.com/158
