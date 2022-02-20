
## @SpringBootApplication

```java
/**
 * Indicates a configuration class that declares one or more @Bean methods 
 * and also triggers auto-configuration and component scanning.
 * This is a convenience annotation that is equivalent to declaring @Configuration, @EnableAutoConfiguration and @ComponentScan.
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
    ...
}
```

<br>

아래 기능을 하는 Configuration Class
1. @Bean 메서드를 선언할 수 있음
2. auto-configuration, component scanning 을 발생시킴

<br>

|Annotation|기능|
|-|-|
|`@SpringBootConfiguration`<br>(`@Configuration`)|@Bean 메서드들을 선언할 수 있음 <br> (오직 한번만 선언 되어야함)|
|`@EnableAutoConfiguration`|AutoConfiguration 클래스(Configuration)들을 설정(로드)|
|`@ComponentScan`|ComponentScan의 base package|


<br><br>

## @SpringBootConfiguration

```java

/**
 * Indicates that a class provides Spring Boot application @Configuration.
 * Can be used as an alternative to the Spring's standard @Configuration annotation 
 * so that configuration can be found automatically (for example in tests).
 *
 * Application should only ever include one @SpringBootConfiguration 
 * and most idiomatic Spring Boot applications will inherit it from @SpringBootApplication.
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
public @interface SpringBootConfiguration {

    ...

}
```

### 요약

- `@Configuration` 과 동일하다.
- 애플리케이션은 오직 하나의 `@SpringBootConfiguration` 을 포함해야 한다.
- `@SpringBootTest` 에서 자동으로 해당 어노테이션(클래스)를 찾는다. 
  - *주석 내용 중, ~~~ configuration can be found automatically (for example in tests)* 

<br><br>

## @EnableAutoConfiguration

```java

/**
 * Enable auto-configuration of the Spring Application Context, 
 * attempting to guess and configure beans that you are likely to need.
 *
 * Auto-configuration classes are usually applied based on your classpath and what beans you have defined.
 *
 * ...
 *
 * When using @SpringBootApplication, the auto-configuration of the context is automatically enabled and adding this annotation has therefore no additional effect.
 *
 * The package of the class that is annotated with @EnableAutoConfiguration,
 * usually via @SpringBootApplication, has specific significance and is often used as a 'default'. 
 * For example, it will be used when scanning for @Entity classes.
 * It is generally recommended that you place @EnableAutoConfiguration (if you're not using @SpringBootApplication) in a root package
 * so that all sub-packages and classes can be searched.
 *
 * Auto-configuration classes are regular Spring @Configuration beans. 
 * They are located using the SpringFactoriesLoader mechanism (keyed against this class). 
 * Generally auto-configuration beans are @Conditional beans (most often using @ConditionalOnClass and @ConditionalOnMissingBean annotations).
 */

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {

	String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

	/**
	 * Exclude specific auto-configuration classes such that they will never be applied.
	 * @return the classes to exclude
	 */
	Class<?>[] exclude() default {};

	/**
	 * Exclude specific auto-configuration class names such that they will never be
	 * applied.
	 * @return the class names to exclude
	 * @since 1.3.0
	 */
	String[] excludeName() default {};

}
```

### 요약

- Spring Application Context 의 auto-configuration 을 활성화
  - AutoConfiguration classes ( + SpringFactoriesLoader, @Configuration beans )
  - 사용자 정의 Beans(classes)
- 해당 어노테이션이 달린 클래스 패키지 (= 'default' 패키지)
  - (하위 경로의) @Entity classes 스캔

> 해당 내용은 따로 자세히 작성(확인)

<br><br>

## @ComponentScan

```java
/**
 * Configures component scanning directives for use with @Configuration classes.
 *
 * ...
 *
 * Either basePackageClasses or basePackages (or its alias value) may be specified to define specific packages to scan. 
 * If specific packages are not defined, scanning will occur from the package of the class that declares this annotation.
  */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Repeatable(ComponentScans.class)
public @interface ComponentScan {

    ...

}
```