컴파일 시점에 Annotation Processor 를 통해 소스코드의 AST(Abstract Syntax Tree)를 동적으로 조작/수정한다.<br>

컴파일 시점에 바이트 코드를 변환하여 추가적인 코드를 주입/생성한다.

<br>

### AnnotationProcessor

컴파일 단계에서 정의/사용된 annotation 코드를 스캔/분석/처리하기 위해 사용되는 훅이다.

컴파일 시점에 끼어들어 annotation 이 붙어있는 코드를 스캔/분석/처리하여 추가적인 소스 코드를 만들어낼 수 있다.

<br>

### 참고

1. [Lombok의 동작원리](https://applefarm.tistory.com/136)
2. [Lombok은 어떻게 동작하는 걸까?](https://jeongcode.github.io/docs/java/Annotation%20Processor/)