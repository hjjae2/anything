> 우선 개념만 정리, 다시 학습 필요 !!

<br>

## LDAP(Lightwieght Directory Access Protocol)

디렉토리 서비스를 제공하기 위한 프로토콜

네트워크상에서 조직(Organization), 개인(Private), 파일(File), 디바이스(Device) 등을 **'찾아볼 수 있게 해주는'** 소프트웨어 프로토콜

LDAP 은 DAP 의 스펙을 최대한 유지 + 경량화 → **네트워크 부담을 줄이고 활용성을 높임**

데이터 형식에 있어서, 대부분 단순한 문자열을 사용 → '구현 단순화', '성능 향상'시켰다고 한다.

> LDAP 등장하기 전, 디렉토리 서비스 표준이었던 X.500 DAP(Directory Access Protocol)이 존재했었다. <br>
> 하지만 이 프로토콜은 무겁고, 제약이 많았다고 한다. 때문에 잘 활용되지 않았다고 한다. 
> 
> DAP 의 경우, OSI 7 Layer 에서 운영된다고 한다. <br>
> 반면에, LDAP 은 TCP/IP 위에서 운영된다고 한다.

<br>

**흐름 예시**

1. `사용자/Application` 에서 LDAP 을 통해 LDAP 서버에 요청
   -  TCP/IP
2. LDAP 서버에서 요청 처리 후 응답

```text
                   TCP/IP
사용자/Application ---------> LDAP Server ---------> DB
```

<br>

**RDB vs LDAP**

\* LDAP 서버도 결국에는 데이터를 저장하는 데이터베이스 유형 중 하나

|서버|설명|
|-|-|
|RDB|- 저장된 여러 데이터를 종합적으로/복합적으로 조회 검색 가능 <br>(LDAP 에 비해) 조회 성능 ↓ <br> (LDAP 에 비해) 쓰기 작업 안정성 ↑ <br><br> 목적 : 복합(읽기, 쓰기) 용도로 사용|
|LDAP|- 저장된 데이터를 신속하고 빠르게 조회/검색 가능 <br>(RDB 에 비해) 조회 성능 ↑ <br> - (RDB 에 비해) 쓰기 작업 안정성 ↓ <br><br> 목적 : 조회/검색 용도로 사용|

<br>

## LDAP 구조(모델)

### Information 모델

- 데이터 종류, 디렉토리에 저장되는 기본 단위에 대한 정의
- 기본 요소 : `Entry`, `Attribute`, `Value`

|용어|설명|
|-|-|
|Entry|디렉토리에서 정보를 표현하는 기본 단위 <br> - Entry 는 다수의 attribute 로 구성 <br> - Entry 는 DN(Distinguish Name)으로 구분 <br> - Tree 구조 형성 (DIT, Directory Information Tree)|
|Attribute|(Entry 안의) 값을 구성하는 단위 <br> - Type, Value 로 구성 <br> - Type 은 Attribute 가 포함하는 정보의 종류를 명시 <br> (단순하게 Key, Value 의 Key 라고 보면 될 것 같다.) <br> - dn(distinguished name), o(organization), cn(common name), c(country), rdn(relative dn), ou(organization unit), sn(surname), ...|
|Value|(Attribute 의) 값 <br> Value 는 Syntax, Matching Rule 을 표현할 수 있다.(?) |

<br>

(앞서 언급한바와 같이)<br>
LDAP 디렉토리 구조는 Entry 데이터들을 트리 구조로 형성, 관리 (DIT, Directory Information Tree)

|용어|설명|예시|
|-|-|-|
|Root Entry(Suffix)|DIT 중 가장 상위에 있는 Entry|
|ObjectClass|Entry 생성 시 Object Class 지정 → Object Class 내의 Attribute 사용 <br><br> 즉, (Entry 내에서) 꼭 필요하거나, 가질 수 있는 Attribute 타입에 대한 정의 <br><br> Entry 내의 클래스(?)라고 볼 수 있을 것 같다.|ObjectClass : cn, sn 정의 (Attribute) <br><br> Entry 생성 시 Person(ObjectClass) 지정 → Entry 에 cn, sn 값 설정 가능|
|Schema|ObjectClass, Attribute 를 정의하는 파일/규칙 <br><br> ObjectClass에 어떤 Attribute 가 들어갈지, Attribute 값에 대한 제약, 조건 등과 관련된 규칙들을 정의||

<br><br>

### Naming 모델

- (LDAP 디렉토리 구조에서) 각 Entity 를 어떻게 식별/참조하고 구성할 것인지에 대한 정의

<br>

|용어|설명|
|-|-|
|DN|RND 값들을 이어 붙여 생성된 고유한 문자 (Distinguished Name) <br><br> - "CN=leehyunjae, OU=People"|
|RDN|각 Entry 의 Attribute (DN 값 중 가장 좌측의 컴포넌트) <br><br> - "OU=People" 에서 OU <br> - "CN=leehyunjae" 에서 CN|


<br><br>

### Functional 모델


- LDAP 디렉토리에서 작업하는 명령어를 다룸
  - 데이터에 접근하는 명령어에 대한 정의
  - 명령어에 따라 3가지 기능(그룹)으로 구분

|그룹|명령어|설명|
|-|-|-|
|**질문 (Interrogation)**|Search|Entry 검색|
|**질문 (Interrogation)**|Comapre|Entry 비교|
|**갱신 (Update)**|Add|Entry 추가|
|**갱신 (Update)**|Modify|Entry 수정|
|**갱신 (Update)**|Delete|Entry 삭제|
|**인증/제어 (Authentication/Control)**|Bind|디렉토리 서버 연결 시, 사용자 인증|
|**인증/제어 (Authentication/Control)**|Unbind|디렉토리 서버와 연결 해제|
|**인증/제어 (Authentication/Control)**|Abandon|이전의 요청한 명령 취소|

<br><br>

### Security 모델

- 디렉토리에 접근하는 사용자 '인증', '인가'를 통해 서비스를 보호하는 방식
  - (TCP 프로토콜로 연결이 이뤄지기 위해) 서버 ←→ 클라이언트 간의 인증 과정 필요
  - '인증'된 경우만 디렉토리의 정보 제공
  - SSL/TLS 적용 가능
- v3 버전 : 기존 보안 방식 + 외부 인증 기능 제공할 수 있는 SASL(Simple Authentication and Security Layer) 방식 제공

> SASL 학습 필요

<br><br>

## LDAP 서버 (종류)

|종류|설명|
|-|-|
|OpenLDAP|LDAP 디렉토리 구축/관리할 수 있는 CLI 기반의 서버 <br><br> LDAP 출시 이후 최초의 오픈소스 (현재까지 꾸준히 활용되고 있음) <br><br> LDAP 기능만 제공 (확장성 ↑)|
|Apache Directory Server|자바로 작성된 LDAP 서버 <br><br> 이클립스와 같은 자바 기반의 응용 프로그램에서 쉽게 임베드할 수 있음 <br><br> LDAP + 다른 기능 포함 (DNS, Kerberos, ...)|
|Active Directory|LDAP + 다양한 형태의 디렉토리 구조 / 여러 프로토콜 제공 <br><br> 기존 LDAP 확장하여 다양한 기능 제공 <br><br> GUI 제공 <br><br> **\*윈도우 환경에서만 서비스 가능**|

<br><br>

## LDAP 클라이언트/API (종류)

> \* Java 기반 클라이언트

|종류|설명|
|-|-|
|JNDI|JDK2 부터 제공된 내장 라이브러리 <br><br> 네이밍, 디렉토리 관련 기능 제공하는 라이브러리 <br><br> * LDAP 이 가진 모든 기능 (Controls), Extension 등)을 제공한다고 한다.|
|Spring LDAP|Spring 프레임워크 기반의 LDAP 라이브러리 <br><br> (JNDI 와 비교해) 간결하고, 쉽게 활용 가능 + 유연한 예외 처리 제공|
|UnboundID LDAP|Ping Identity 사에서 제공 <br><br> 다른 LDAP 라이브러리보다 빠르고, 쉽다(고 강조하고 있다고 한다.)<br><br> 데이터 포맷, 암호화 등을 비롯한 다양한 API 들과 LDAP 기반의 In-Memory 테스트 환경 제공|

<br><br>

## 참고

1. https://ldap.or.kr/ldap-%EC%9D%B4%EB%9E%80/
2. https://www.samsungsds.com/kr/insights/LDAP.html