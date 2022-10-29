## IaC의 장점 

- Infrastructure 버전을 관리할 수 있다.
- Infrastructure 선언적으로 관리할 수 있다.
- Infrastructure 여러 벌(set) 배포할 수 있다.
- 각 리소스의 필수 요소들을 파악하기 쉽다.
  - 리소스가 대체될 것이라는 warning 문구를 표시해주기도 하고, replace 할 것인지 확인 여부를 묻는 기능도 있다. (approval 하면 실제 배포되는 것)
- 예상치 못한 과금을 예방할 수 있다.
- 비즈니스 로직 작성에 집중할 수 있다.

<br>

![](../../images/[데브옵스]%20CDK%20for%20Terraform으로%20나만의%20AWS%20Resource%20정의하기_24.png)

![](../../images/[데브옵스]%20CDK%20for%20Terraform으로%20나만의%20AWS%20Resource%20정의하기_51.png)

<br>

## CDK for TF 

1. AWS CDK 개발진이 Hashi Corp와 함께 개발한 IaC Kit
2. Generally Available at 2022.08.09
3. ts, python, java, c#, go 로 infrastructure 정의, 배포, 운영 가능하다.
4. CDKTF와 AWS CDK는 호환성을 보장하려고 한다.


<br>

### 왜 CDK for TF 를 사용해야할까?

콘솔(AWS Management Console) 관리의 경우,
1. 쉽다.
2. 작업 기록이 남지 않는다.
3. 규모가 커질수록 운영이 어렵다.

<br>

AWS CLI & SDK, AWS CloudFormation 의 경우,

1. 작업 기록을 남길 수 있다.
2. 어렵다.
3. 작업량이 많다.
4. 개발자 친화적이지 않다.

<br>

AWS CDK 의 경우,
1. 쉽다.
2. 작업량이 적다.
3. 개발자 친화적이다.

<br>

**CDK for TF 의 경우,**

1. AWS CDK + TF 장점(TF 의 경우, docker, k8s 등 AWS 외의 리소스도 관리할 수 있다.)
2. TF ecosystem 활용 가능하다.

<br>

### CDKTF Building Blocks

![](../../images/[데브옵스]%20CDK%20for%20Terraform으로%20나만의%20AWS%20Resource%20정의하기_03.png)

![](../../images/[데브옵스]%20CDK%20for%20Terraform으로%20나만의%20AWS%20Resource%20정의하기_44.png)

![](../../images/[데브옵스]%20CDK%20for%20Terraform으로%20나만의%20AWS%20Resource%20정의하기_46.png)

> Resource = AWS 리소스

<br>

### CDKTF와 AWS CDK를 관통하는 개념 : Construct

1. CDK 프로젝트를 설명할 수 있는 용어
2. 모든 구성요소는 Construct의 하위 클래스
3. App, Stack, Resource 모두 Construct이다.
4. 추상화 정도에 따라서 Construct Type 이 존재
   - L1: single resource 수준의 configuration e.g.) EC2 하나 생성
   - L2: 하나 이상의 resource와 추가적인 기능이 포함된 configuration e.g.) S3 Bucket 생성 후 업로드 메서드 구현
   - L3: 배포, 호스팅에 필요한 모든 resource 를 포함한 configuration


<br>

### Demo

![](../../images/[데브옵스]%20CDK%20for%20Terraform으로%20나만의%20AWS%20Resource%20정의하기_00.png)

