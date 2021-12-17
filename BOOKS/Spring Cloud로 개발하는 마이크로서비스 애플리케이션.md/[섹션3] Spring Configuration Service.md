서버, 클라이언트 구성에 필요한 설정 정보(application.yml)를 외부 시스템에서 관리할 수 있다. 

하나의 중앙화 된 저장소(GIT Repository, Secure Valut, Secure File Storage 등)에서 관리할 수 있다.

각 서비스를 다시 빌드하지 않고 바로 적용 가능하다.

애플리케이션 배포 파이프라인을 통해 DEV-STAGE-PROD 환경에 맞는 구성 정보로 사용 가능하다.

저장소로부터 Spring Cloud Config Server가 config 파일(값)을 가져와 각각의 service 에 전달해줄 수 있다.

### Config Server

```java
@SpringBootApplication
@EnableConfigServer
public class ConfigServiceApplication {
    ...
}
```

```yaml
server:
    port: 8888

spring:
    application:
        name: config-service
    cloud:
        config:
            server:
                git:
                    uri: file:///Users/leehyunjae/local-repository
# spring.cloud.config.server.git.uri 에 로컬의 repository 경로를 작성할 수 있다.
```

- http://127.0.0.1:8888/ecommerce/default
- http://127.0.0.1:8888/ecommerce/dev
- http://127.0.0.1:8888/ecommerce/prod

<br>

### Config Client

Dependencies 추가

- spring-cloud-starter-config
- spring-cloud-starter-bootstrap (or spring.cloud.bootstrap.enabled=true)

**bootstrap.yml**
 
```yaml
spring:
    cloud:
        config:
            uri: http://127.0.0.1:8888 # Config Server URI
            name: ecommerce
```

> bootstrap.yml 의 우선순위가 더 높다. (bootstrap.yml > application.yml)

application.yml 파일에 해당 내용을 작성해도 된다. 하지만 지금은 application.yml 을 config server 로 관리(즉, application.yml 을 갖고 있지 않을 예정)하고자 하기 때문에 bootstrap.yml 에 작성한다.