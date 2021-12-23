## Vue Life-cycle

1. new Vue() 
2. Init (Event, Lifecycle) : 이벤트 & 라이프사이클 초기화
3. **beforeCreate**
4. Init (injections & reactivity) : 반응성 주입
5. **created**
6. ... (el, template 속성 확인 등)
7. **beforeMount**
8. **mounted**
9. **beforeUpdate**
10. **updated**
11. **beforeDestroy**
12. **destroyed**

<br>

### beforeCreate()

가장 먼저 실행되는 훅이다.

Vue 인스턴스가 생성된 직후에 실행된다.

데이터(`data`), 이벤트(`methods`) 속성이 정의되어 있지 않다. 

화면(template) 요소에 접근할 수 없다.

<br>

### created()

데이터(`data`)와 이벤트(`methods`)가 초기화되어, 접근할 수 있다.

화면(template) 요소에 접근할 수 없다.

<br>

### beforeMount()

render() 함수가 실행되기 직전의 단계이다. 즉 화면에 렌더링 되기 전에 실행된다.

컴포넌트에 접근할 수 있다.

$el 속성을 사용할 수 없다.

<br>

### mounted()

렌더링되고 난 직후 호출된다.

$el 속성을 사용할 수 있다.

대게 제일 많이 사용되는 훅이다.

<br>

### beforeUpdate()

데이터 변경을 감지하고, re-render 전에 호출된다.

<br>

## updated()

re-render 후에 호출된다.

<br>

### beforeDestroy()

인스턴스가 삭제되기 전에 호출된다.

<br>

### destroyed()

Vue 인스턴스가 모두 삭제된 후 실행된다.

Vue 인스턴스에 정의된 모든 속성, 이벤트 등이 사라진다.


---

### 생성 단계 (Create)

- `beforeCreate()`, `create()` 단계가 포함된다.
- 이벤트, 생명주기 메소드가 초기화된다.


### 초기화 단계 (Mount)

- Vue 문법을 적용, 화면에 값들을 렌더링한다.

### 갱신 단계 (Update)

- 데이터의 변화를 감지하면 화면을 다시 렌더링(re-render)한다.
- Virtual DOM 을 활용하여 렌더링한다.

### 소멸 단계 (Destroy)

- 이벤트 등을 해제하고, Vue 인스턴스를 메모리에서 해제한다.


<br>

---
### 참고
1. https://hyeooona825.tistory.com/40