## Top 10 Architecture Characteristics

> 참고 : https://medium.com/@abd0hrz/top-10-architecture-characteristics-non-functional-requirements-with-cheatsheet-f639458d357d


## 1. Scalability

Scalability is a achievable with horizontal/vertical scaling of the machine.

<br>

### Traffic Pattern

**Understand the traffic pattern of the system.** :star:

|Pattern||
|-|-|
|Diurnal Pattern|Traffic increases in the morning and decreases in the evening for a particular region.|
|Global / Regional|Regional Heavy usage of the application.|
|Thundering Herd|These could occur during peak time or in densely populated areas.|

> 우리 시스템의 트래픽 패턴을 이해하자. :star:
> 
> 1. 일반적인 패턴(낮에 트래픽이 증가하고 저녁에 감소하는 패턴)
> 2. 지역 패턴(지역 별로 트래픽이 집중되고, 집중되지 않는 패턴)
> 3. Thundering Herd (피크 타임, 지역에 의해 요청이 급격하게 증가하는 패턴)

<br>

### Elasticity

Ability to quickly **spawn** a few machines (for handling) and **shrink** when the demand is reducing.

<br>

### Latency

Ability to serve the request as quickly as possible.

<br>

## 2. Availability

The proportion of time that a system is functional and working.

It is measured as a percentage of uptime.

<br>

## 3. Extensibility

The ability to extend a system.

The cost about effort required to implement the extension.

### Modular / Reusability

> 생략

### Pluggability

Ability to easily plug in/out.

> (개인적으로) 최근에 이 Pluggability 측면에 대해서 고민했다.

<br>

## 4. Consistency

> 생략

<br>

## 5. Resiliency

The ability to **handle and recover** from accidental and malicious failures.

### Recoverability

- **DR(Disaster Recovery)** : DR consists of best practices designed to prevent or minimise data loss and business disruption resulting from catastrophic events.

### Design Patterns

- **BulkHead** : Isloate elements of an application into pools so that if one fails, the others will contine to function. (**장애를 고립시키자.**)
- **Circuit Breaker**
- **Leader Election** : Elect leader instance that can coordinates, manages other instances.

<br>

## 6. Usability

- **Accessibility** : Make the software available to people with the broadest range of characteristics and capabilities.
- **Learnability** : How easy users can learn how to use the software?
- **API Contract** : API Contracts can help to understand easily

<br>

## 7. Observability

The ability to collect data bout program execution, internal states of modules, and communication between components.

To improve this, use various logging and tracing techniques, tools.

- **Logging** : event logs, transaction logs, message logs, server logs, ...
- **Alerts & Monitoring**
- **L1 / L2 / L3** : Setup on-call support process for L1 / L2.


> L1 / L2 / L3 프로세스는 흥미롭다. :star:
>  
> 찾아봐보자.

## 8. Security

auditability, legality, authentication, authorization, ...

<br>

## 9. Durability

- Replication
- Fault Tolerance
- Archivability

<br>

## 10. Agility

> It has become today’s buzzword when describing a contemporary software method.

- Maintainability
- Deployability
- Configurability

<br>

## Conclusion

모든 아키텍처의 특징이 필요한 건 아니다. (반대로 모두 필요할 수도 있다.)

이는 개발자(아키텍터)가 현재 프로젝트에 맞게, 기능 요구 사항에 맞게 적절하게 선택하고 설계할 수 있어야 한다.

다음과 같은 질문들을 던져봐도 좋을 것이다.

- 어느 정도 처리량이 필요한지?
- 보안적 요구 사항은 없는지?
- 코드 유지보수(기능 추가, 변경, 삭제)는 쉬운지?
- ...