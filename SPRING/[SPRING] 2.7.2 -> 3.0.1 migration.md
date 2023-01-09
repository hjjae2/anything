## 전제 조건

- Java 17 이상
- SpringBoot 2.7.x

### 참고 문서

- [wiki: Spring Boot 3.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0-Migration-Guide)

<br>

## TIP

다음 라이브러리가 도움을 준다.

```
runtime("org.springframework.boot:spring-boot-properties-migrator")
```

의존성 설치 후 애플리케이션을 시작하면, 로그에 변경된 사항을 알려준다.

![](../images/[SPRING]%202.7.2%20->%203.0.1%20migration_33.png)

<br>

## Core Changes

### [Jakarta EE](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0-Migration-Guide#jakarta-ee) (패키지명 변경 : `javax` -> `jakarta`)

> history 잠깐 보면, 2017년 오라클에서 이클립스 재단으로 자바 EE를 이관했다. (*"썬 마이크로시스템즈를 인수한 오라클이 사실상 자바EE의 수익화에 실패하면서 기술 주도권을 포기한 것으로 판단됩니다."*)
> (오픈소스) 이클립스 재단으로 이관된 자바 EE의 공식 명칭이 자카르타 EE 이다. (*"이클립스 재단으로 이관된 자바EE의 공식 명칭은 자카르타EE, 프로젝트 명은 EE4J(Eclipse Enterprise for Java)로 변경되었습니다."*)
> 
> *"오라클이 자바EE 프로젝트는 이관했지만 자바 상표권은 여전히 보유하고 있기 때문에 자바 네임스페이스 사용에 제약이 있었습니다. 이러한 이유로 자카르타EE에서는 자바 네임스페이스가 Jakarta로, API 패키지명은 javax.\* 에서 Jakarta.\* 로 변경되었습니다."*
>
> 참고 : https://www.samsungsds.com/kr/insights/java_jakarta.html

<br>

### [@ConstructingBinding No Longer Needed at the Type Level](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0-Migration-Guide#constructingbinding-no-longer-needed-at-the-type-level)

@ConstructorBinding is no longer needed at the type level on @ConfigurationProperties classes and should be removed. When a class or record has multiple constructors, it may still be used on a constructor to indicate which one should be used for property binding.

> 클래스 레벨에 사용하는 것은 안되고, 생성자 위에 사용하는 것이 가능해졌나보다.

```kotlin
// before
@Target({ ElementType.TYPE, ElementType.CONSTRUCTOR })
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface ConstructorBinding {

}

// after
@Target({ ElementType.CONSTRUCTOR, ElementType.ANNOTATION_TYPE })
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface ConstructorBinding {

}
```

<br>

### [Auto-configuration Files](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0-Migration-Guide#auto-configuration-files)

2.7에서 auto-configuration 위해 `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` 이 도입되었다. (예시 - https://github.com/spring-projects/spring-boot/blob/main/spring-boot-project/spring-boot-autoconfigure/src/main/resources/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports)

기존에 spring.factories(`org.springframework.boot.autoconfigure.EnableAutoConfiguration`)와 호환 가능했지만, 이번 릴리즈 부터는 spring.factories 는 제거됐다.

<br>

## [Data Access Changes](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0-Migration-Guide#data-access-changes)

### (application.yml) `spring.redis` -> `spring.data.redis` 설정 변경

https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0-Migration-Guide#redis-properties


<br>

## [Spring Batch Changes (5.0)](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0-Migration-Guide#spring-batch-changes)


### [@EnableBatchProcessing is now discouraged](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0-Migration-Guide#enablebatchprocessing-is-now-discouraged)

Previously, @EnableBatchProcessing could be used to enable Spring Boot’s auto-configuration of Spring Batch. It is no longer required and should be removed from applications that want to use Boot’s auto-configuration. A bean that is annotated with @EnableBatchProcessing or that extends Batch’s DefaultBatchConfiguration can now be defined to tell the auto-configuration to back off, allowing the application to take complete control of how Batch is configured.

\+ https://stackoverflow.com/a/74845329


<br>

### [Multiple Batch Jobs](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0-Migration-Guide#multiple-batch-jobs)

<br>

### [JobBuilderFactory and StepBuilderFactory bean exposure/configuration](https://github.com/spring-projects/spring-batch/wiki/Spring-Batch-5.0-Migration-Guide#jobbuilderfactory-and-stepbuilderfactory-bean-exposureconfiguration)

**jobBuilderFactory**

```kotlin
// before
@Bean
fun myJob(): Job = jobBuilderFactory
    .get("myJob")
    .start(myStep())
    .build()
```
```kotlin
// after
@Bean
fun myJob(): Job = JobBuilder("myJob", jobRepository)
    .start(myStep())
    .build()
```

JobBuilder, StepBuilder 는 원래 있었는데 생성자 하나가 더 생겼다. (기존 생성자는 `Deprecated`)

```java
@Deprecated(since = "5.0")
public JobBuilder(String name) {
    super(name);
}

public JobBuilder(String name, JobRepository jobRepository) {
    super(name);
    super.repository(jobRepository);
}
```

```java
@Deprecated(since = "5.0")
public StepBuilder(String name) {
    super(name);
}

public StepBuilder(String name, JobRepository jobRepository) {
    super(name);
    super.repository(jobRepository);
}
```

**stepBuilderFactory**

```kotlin
// before
@Bean
fun myStep(): Step = stepBuilderFactory
    .get("myStep")
    .tasklet { _: StepContribution, _: ChunkContext ->
        ...
    }
    .build()
```

```kotlin
// after
@Bean
fun myStep(): Step = StepBuilder("myStep", jobRepository)
    .tasklet(myTasklet, transactionManager)
```

<br>

### [Database schema updates](https://github.com/spring-projects/spring-batch/wiki/Spring-Batch-5.0-Migration-Guide#database-schema-updates)

스키마 변경이 있다. (스키마 마이그레이션 시) 해당 프레임워크 내에 쿼리를 제공하고 있으니 그걸 사용하면 된다.

### [Transaction manager bean exposure/configuration](https://github.com/spring-projects/spring-batch/wiki/Spring-Batch-5.0-Migration-Guide)

(몇 가지 이슈가 있어서) `@EnableBatchProcessing` 가 이제 txmanager 빈을 노출시키지 않음 -> 사용자가 수동으로 설정하게 되었다. (?) :thinking:

```kotlin
// Sample with v4
@Configuration
@EnableBatchProcessing
public class MyStepConfig {

    @Autowired
    private StepBuilderFactory stepBuilderFactory;

    @Bean
    public Step myStep() {
        return this.stepBuilderFactory.get("myStep")
                .tasklet(..) // or .chunk()
                .build();
    }

}
```
```kotlin
// Sample with v5
@Configuration
@EnableBatchProcessing
public class MyStepConfig {

    @Bean
    public Tasklet myTasklet() {
       return new MyTasklet();
    }

    @Bean
    public Step myStep(JobRepository jobRepository, Tasklet myTasklet, PlatformTransactionManager transactionManager) {
        return new StepBuilder("myStep", jobRepository)
                .tasklet(myTasklet, transactionManager) // or .chunk(chunkSize, transactionManager)
                .build();
    }

}
```

```kotlin
@Deprecated(since = "5.0")
public TaskletStepBuilder tasklet(Tasklet tasklet) {
    return new TaskletStepBuilder(this).tasklet(tasklet);
}

public TaskletStepBuilder tasklet(Tasklet tasklet, PlatformTransactionManager transactionManager) {
    return new TaskletStepBuilder(this).tasklet(tasklet, transactionManager);
}
```

> (kotlin의 경우) 람다를 못쓴다. (...?) :thinking:


<br>

## ETC

<br>

### `org.hibernate.dialect.MySQL57Dialect` -> `org.hibernate.dialect.MySQLDialect`

<br>

### jakarta.persistence.sharedCache.mode 설정