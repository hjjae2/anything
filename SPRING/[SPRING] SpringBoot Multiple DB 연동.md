---
layout: post
title: "SpringBoot :: Multiple DB 연동하기"
author: "leehyunjae"
tags: ["springboot", "java"]
---

> SpringBoot 에서 여러 개의 DB 연동해보기

<br>

### 개요

회사에서 새로운 프로젝트를 SpringBoot 환경으로 진행하게 되었다. 기존의 대부분의 프로젝트는 PHP 진영의 CodeIgniter(이하 CI) 라는 프레임워크가 사용되어왔다. SpringBoot로 컨버팅하는 작업도 일부 포함되어 있었기에, 이것저것 알아봐야하는 것들이 많았다.

그 중에 가장 처음 직면한 문제는 <u>`DB 연동` 문제</u>였다. 우리 팀은 기본적으로 Oracle DB를 베이스로 하는데, 몇몇 프로젝트에서는 Mysql 을 사용한 프로젝트도 있었다. 문제는 이번에 작업해야 할 프로젝트가 <u>Oracle, Mysql DB를 둘 다 연동/사용해야 했다.</u>

이번 글에서는 여러 개의 DB 를 연결하기 위해 찾아본 내용을 정리해보려고 한다.

<br>

### 설정

**여러 개의 DB 연결 시, 참고해야 할 것은 다음과 같다.**

**1. 수동으로 설정해주어야 한다!**<br>
 단일 DB 연동시에는 `application.yml`, `application.properties` 의 설정을 통해 자동으로 연결해왔다면, 여러 개의 DB 연동시에는 수동으로 설정해줘야하는 부분이 있다. 예를 들어, 아래와 같은 것들을 설정해주어야 한다.
 - Target package 설정, Entity 설정
 - DB(Datasource) 설정
 - Hibernate 설정 등

**2. (어떤 DB를 적용할지는) Package 를 통해 구분한다.**<br>
 여러 개의 DB를 연동한다는 것은, Repository 마다 연결되어야 할 DB가 다름을 의미할 것이다. 이때 Repository 가 어떤 DB에 연동되어야 할지 결정해줘야 한다. 이것을 Package 경로를 통해 지정해준다. (1번에서 언급된 Target Package, Entity 설정)

<br>

**application.yml 예시**

```yaml
// 기존(단일 DB 연동 시)

spring:
  datasource:
    url: DB's URL
    username: DB's username
    password: DB's password
    driver-class-name: com.mysql.cj.jdbc.Driver
  jpa:
    properties:
      hibernate:
        ~~~
    database: oracle
    database-platform: org.hibernate.dialect.Oracle10gDialect
```

<br>

```yaml
// 변경(여러 개의 DB 연동 시)

datasource:
  mysql:
    jdbc-url: ...
    username: ...
    password: ...
    database: mysql
    database-platform: org.hibernate.dialect.MySQL5InnoDBDialect
    driver-class-name: com.mysql.cj.jdbc.Driver
    ...
  oracle:
    jdbc-url: ...
    username: ...
    password: ...
    database: oracle
    database-platform: org.hibernate.dialect.Oracle10gDialect
    driver-class-name: oracle.jdbc.driver.OracleDriver
    ...
```

앞서 말한 것 처럼, 위의 `spring.datasource` 를 통한 설정/연결을 사용하지 않는다고 보면 된다. 그렇기에 자유로운 key 값으로 config 관련 값을 적어놓는다.

<br>

**Java configuration 설정**

```java
{% raw %}
@RequiredArgsConstructor
@EnableJpaRepositories(
        entityManagerFactoryRef = "firstEntityManager",
        transactionManagerRef = "firstTransactionManager",
        basePackages = {"com.demo.project.repository.first"}
)
@ConfigurationProperties(prefix = "datasource.oracle")
@PropertySource({"classpath:application.yml"})
@Configuration
public class FirstJpaConfiguration extends HikariConfig {

    private final Environment env;

    private String[] packagesToScan = {"com.demo.project.entity.first"};
    ...

    @Bean
    public DataSource firstDataSource() {
        return new LazyConnectionDataSourceProxy(new HikariDataSource(this));
    }

    @Bean
    public LocalContainerEntityManagerFactoryBean firstEntityManager() {
        LocalContainerEntityManagerFactoryBean entityManagerFactoryBean = new LocalContainerEntityManagerFactoryBean();

        entityManagerFactoryBean.setDataSource(firstDataSource());
        entityManagerFactoryBean.setPackagesToScan(packagesToScan);
        entityManagerFactoryBean.setJpaVendorAdapter(new HibernateJpaVendorAdapter());
        entityManagerFactoryBean.setJpaPropertyMap(new HashMap<>() {{
            put("hibernate.dialect", env.getProperty(dialect));
            ...
        }});

        return entityManagerFactoryBean;
    }

    @Bean
    public PlatformTransactionManager firstTransactionManager() {
        JpaTransactionManager transactionManager = new JpaTransactionManager();

        transactionManager.setEntityManagerFactory(firstEntityManager().getObject());

        return transactionManager;
    }
}
{% endraw %}
```

<br>

위의 내용을 한줄한줄 살펴보면,

```java
    ...

    @Bean
    public DataSource firstDataSource() {
        return new LazyConnectionDataSourceProxy(new HikariDataSource(this));
    }
```
첫 번째로는 연결할 DB(DataSource) Bean 을 생성(설정)하는 것이다. (DataSource Bean 을 생성하는 다른 방법도 있다.)

<br>

```java
{% raw %}
    ...

    @Bean
    public LocalContainerEntityManagerFactoryBean firstEntityManager() {
        LocalContainerEntityManagerFactoryBean entityManagerFactoryBean = new LocalContainerEntityManagerFactoryBean();

        entityManagerFactoryBean.setDataSource(firstDataSource());
        entityManagerFactoryBean.setPackagesToScan(packagesToScan);
        entityManagerFactoryBean.setJpaVendorAdapter(new HibernateJpaVendorAdapter());
        entityManagerFactoryBean.setJpaPropertyMap(new HashMap<>() {{
            put("hibernate.dialect", env.getProperty(dialect));
            ...
        }});

        return entityManagerFactoryBean;
    }
{% endraw %}
```

두 번째로는 JPA 관련 설정(EntityManager)을 하는 것이다. DB(DataSource), PacagesToScan, JpaVendor(Hibernate), 이 외 JPA 관련 설정 등을 설정해준다.

<br>

```java
    ...

    @Bean
    public PlatformTransactionManager firstTransactionManager() {
        JpaTransactionManager transactionManager = new JpaTransactionManager();

        transactionManager.setEntityManagerFactory(firstEntityManager().getObject());

        return transactionManager;
    }
```

세 번째로는 TransactionManager 를 설정해주는 것이다. TransactionManager 는 트랜잭션 관리를 위한 객체라고 보면된다. 

> *" 데이타가 문제가 없다면 Commit 을 통해 데이타를 영속화 시키고, 문제가 발생된다면 Rollback 을 통해 트랜잭션의 원자성을 보장해 주는 역할을 담당합니다. "*

즉, 이 객체가 설정되어야 정상적으로 트랜잭션이 관리된다는 것이다. (`@Transactional` 이라는 어노테이션을 많이 사용하는데, 이 어노테이션이 적용되어 있는 부분에 대해서 TransactionManager 가 관리해주고 있는 것이다.)

원래 Spring의 기본 TransactionManager는 DataSourceTransactionManager(구현체)를 사용한다고 한다. 우리는 JPA를 사용할 것이기 때문에 JpaTransactionManager(구현체)를 사용한다.

> *" TransactionManager 인 DataSourceTransactionManager 구현체를 사용하지 않고 JpaTransactionManager 구현체를 사용한다는 점입니다. "*

<br>

### 요약

1. DB 별 DataSource Bean을 생성/설정한다.
2. DB 별 JPA(EntityManager, LocalContainerEntityManagerFactoryBean)을 설정한다.<br>
이때 1번에서 생성한 DataSource Bean 을 주입해준다.
3. DB 별 TransactionManager(PlatformTransactionManager)을 설정한다.<br>
이때 2번에서 생성한 EntityManager Bean 을 주입해준다. 해당 EntityManager의 트랜잭션을 관리하겠다는 의미이다.

<br>

### 참고

- [JPA 다중 데이터소스 관리](https://jogeum.net/2)
- [What transaction manager to use? (JPA, Spring)](https://stackoverflow.com/questions/3880563/what-transaction-manager-to-use-jpa-spring)