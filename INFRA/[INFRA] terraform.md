# Terraform

**Infrastructure as Code** <br>
( = Infrastructure 관리 도구 )

<br><br>

## tfenv

테라폼 버전 매니저

> nvm 같은 버전 매니저

<br><br>

## 기본 개념

**프로비저닝**

프로세스, 서비스를 실행하기 위한 준비 단계

- 네트워크, 컴퓨팅 자원 준비 작업
- (준비된 컴퓨팅 자원에) 사이트 패키지, 애플리케이션 의존성 준비 작업

명확한 경계는 불분명하지만 테라폼은 주로 전자(네트워크, 컴퓨팅 자원 준비)를 주로 다룬다.

<br>

**프로바이더**

테라폼 ⇿ 외부 서비스(프로바이더) 연결해주는 모듈
- 예를 들어, 테라폼으로 AWS 컴퓨팅 자원을 생성한다면 'aws 프로바이더'

|프로바이더 종류|분류|
|-|-|
|AWS|클라우드 서비스|
|GCP|클라우드 서비스|
|Azure|클라우드 서비스|
|Github|특정 기능을 제공하는 서비스|
|Datadog|특정 기능을 제공하는 서비스|
|DNSimple|특정 기능을 제공하는 서비스|
|MySQL|로컬 서비스|
|RabbitMQ|로컬 서비스|
|Docker|로컬 서비스|
|...|...|

<br>

**리소스(자원)**

프로바이더가 제공해주는 조작 가능한 대상의 최소 단위
- AWS 프로바이더는 aws_instance 리소스 타입을 제공 → EC2 가상 머신 리소스를 선언/조작 가능
  - EC2 인스턴스, 시큐리티 그룹, 키 페어 등

<br>

**HCL**

테라폼에서 사용하는 설정 언어
- 모든 설정, 리소스 선언은 HCL 을 사용
- HCL 파일 확장자 : `.tf`

<br>

**계획(Plan)**

테라폼 프로젝트 디렉토리 아래의 모든 `.tf` 파일 내용에 대한 작업 계획 확인 (+ 실제로 적용 가능한지 확인)

- 명령어 : `terraform plan`
- 어떤 리소스가 생성, 수정, 삭제될 지에 대한 계획 확인

<br>

**적용(Apply)**

테라폼 프로젝트 디렉토리 아래의 모든 `.tf` 파일 내용 적용

- 명령어 : `terraform apply`
- 리소스 생성, 수정, 삭제 작업 적용

<br><br>

## 단계 요약

**1. `.tf` 파일 작성 (with HCL) : 리소스 선언** <br>
**2. plan : 리소스 계획 확인 (`terraform plan`)** <br>
**3. apply : 리소스 적용 (`terraform apply`)** <br>

\+ destory : 리소스 제거

> 이후에는 [테라폼(Terraform) 기초 튜토리얼 : AWS 프로바이더 정의](https://www.44bits.io/ko/post/terraform_introduction_infrastrucute_as_code#%EB%91%90-%EB%B2%88%EC%A7%B8-%EB%8B%A8%EA%B3%84---hcl%EB%A1%9C-%EB%A6%AC%EC%86%8C%EC%8A%A4-%EC%A0%95%EC%9D%98%ED%95%98%EA%B3%A0-aws%EC%97%90-%ED%94%84%EB%A1%9C%EB%B9%84%EC%A0%80%EB%8B%9D) 페이지의 실습 부분에서 상세히 확인할 수 있다.

<br>

## 참고

- [테라폼(Terraform) 기초 튜토리얼](https://www.44bits.io/ko/post/terraform_introduction_infrastrucute_as_code)
- [좌충우돌 Terraform 입문기](https://techblog.woowahan.com/2646/)