## :zap: compilePath & runtimeClasspath

<img src="../images/[GRADLE]%20Basic_50.png">

<br><br>

## :zap: testCompilePath & testRuntimeClasspath

<img src="../images/[GRADLE]%20Basic_51.png">

<br>

> *" Configuration inheritance is heavily used by Gradle core plugins like the Java plugin. For example the testImplementation configuration extends the implementation configuration. "*
> 
> 출처 : [Configuration inheritance and composition](https://docs.gradle.org/current/userguide/declaring_dependencies.html)


<br><br>

## :zap: example

```sh
dependencies {
    ...
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'

    ...

    compileOnly 'org.projectlombok:lombok'

    ...

    runtimeOnly 'com.h2database:h2'
    runtimeOnly 'mysql:mysql-connector-java'

    ...

    annotationProcessor 'org.projectlombok:lombok'

    ...

    testImplementation 'org.springframework.security:spring-security-test'

    ...
}
```

<br><br>

### 참고

- [[Spring] Gradle 파일 implementation, api, runtimeOnly, compileOnly... 등에 대해](https://bepoz-study-diary.tistory.com/372)
- [[Gradle] build.gradle의 dependencies 블록 한 번에 정리하기. implementation, testImplementation의 차이와 라이브러리 구성](https://kotlinworld.com/316)