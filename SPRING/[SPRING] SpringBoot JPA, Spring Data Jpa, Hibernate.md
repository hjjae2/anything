---
layout: post
title: "SpringBoot :: JPA? Spring Data JPA? Hibernate?"
author: "leehyunjae"
tags: ["springboot", "java", "jpa"]
---

> [jpa vs hibernate vs spring data jpa](https://suhwan.dev/2019/02/24/jpa-vs-hibernate-vs-spring-data-jpa/) 글을 읽고 이해한 내용 정리해보기

<br>

### JPA (Java Persisetence API)

JPA 는 인터페이스(명세)이다. 간단하게 **"관계형 데이터베이스 사용을 위한 인터페이스"** 이다. 

`EntityManager` 가 바로 JPA 인터페이스의 실물(구현체)이다.

`EntityManager` 를 살펴보면 관계형 데이터베이스 사용을 위한 기능들이 정의되어 있음을 알 수 있다.

```java
public interface EntityManager {

    /**
     * Make an instance managed and persistent.
     * ...
     */
    public void persist(Object entity);
    
    /**
     * Merge the state of the given entity into the
     * current persistence context.
     * ...
     */    
    public <T> T merge(T entity);

    ...
```

<br>

### JPA 의 구현체

JPA는 인터페이스라고 했다. 인터페이스이니까 분명 구현체가 존재할 것이다. 그 구현체는 어떤 클래스일까.

그것이 바로 `Hibernate` 이다. 조금 더 구체적으로는 `Hibernate` 의 `Session(SessionImpl)` 클래스가 `EntityManager` 를 구현하고 있다. 

> 클래스의 이름이 Session 이어서 이게 맞나 싶었는데, 블로그 글에서 정확한 그림이 있었다.<br>
> 참고로 Hibernate 가 아닌 다른 구현체를 사용할 수도 있다. 블로그 글에서는 DataNucleus, EclipseLink 와 같은 구현체들이 있다고 한다.

```
(Hibernate)Session(SessionImpl) ---> EntityManager

(Hibernate)SessionFactory(SessionFactoryImpl) ---> EntityManagerFactory

(Hibernate)Transaction(TransactionImpl) ---> EntityTransaction
```

<br>

### Spring Data JPA

Spring Data JPA는 JPA를 사용하기 쉽게 만들어 놓은 모듈이라고 한다. 즉, 한단계 더 추상화시켜 개발자가 사용하기 쉽게 만들어놓은 것이다. 우리가 JPA 를 직접적으로 사용했다면 아래와 같이 `EntityManager` 를 직접적으로 사용했을 것이다.

```java
@Autowired
EntityManager em;

...

public void test() {
    Entity entity = new Entity(...);

    em.persist(entity);

    ...
}
```

<br>

그러나 대부분의 경우에 EntityManager 를 직접적으로 사용하지는 않았을 것이다. 대신 `Repository` 라는 것을 사용하고 있다.

```java
@Autowired
EntityRepository repo;

...

public void test() {
    Entity entity = new Entity(...);

    repo.save(entity);

    ...
}
```

> " 사용자가 Repository 인터페이스에 정해진 규칙대로 메소드를 입력하면, Spring이 알아서 해당 메소드 이름에 적합한 쿼리를 날리는 구현체를 만들어서 Bean으로 등록해준다. "

즉, 우리가 Repository 를 규칙에 맞게 작성하면 Spring은 이 Repository 인터페이스의 구현체를 만들어주고 Bean 으로 등록해준다는 것이다. 또 이 구현체에서는 JPA(EntityManager)를 사용할 것이다.

<br>

```
[Application]

    ↕

[Spring Data JPA]
    JPA
    Hibernate(DataNucleus, EclipseLink, ...)
    JDBC

    ↕

[Database]
```