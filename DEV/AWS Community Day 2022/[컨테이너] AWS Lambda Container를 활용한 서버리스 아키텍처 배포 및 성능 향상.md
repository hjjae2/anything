## Introduction

- Fargate
- Kinesis (Data Firehost, Stream)
- S3
- SQS
- Lambda
- Step Functions
  - 구조화된 Lambda workflow 를 구성하고 있다.

<br>

## Why Lambda Container?

Data 처리 & Lambda 사용에 있어서 용량 제한으로 인해 다양한 라이브러리 사용에 제한이 있었다. 라이브러리 용량이 조금만 커도 사용에 제한이 있었다.

기존 labmda 기반 아키텍처를 유지하면서 conatiner 기반으로 변경할 수 있어야 했다.

![](../../images/[컨테이너]%20AWS%20Lambda%20Container를%20활용한%20서버리스%20아키텍처%20배포%20및%20성능%20향상_58.png)

![](../../images/[컨테이너]%20AWS%20Lambda%20Container를%20활용한%20서버리스%20아키텍처%20배포%20및%20성능%20향상_32.png)

<br><br>

## Lambda Container with Serverless Framework

도커 이미지 필요하다. AWS 에서 제공하는 이미지를 사용하는게 제일 속 편하다. (이 경우가 아니면 추가 설정이 들어갈 수 있다고 한다.)

serverless framework의 provider 설정을 통해 ecr 에 배포할 수 있다.

<br>

**장점**

1. 용량 문제 해소
2. 하나의 이미지를 통해 다용도(API server, Lambda 등 다양한 곳에서 같은 이미지)로 사용 가능했다.
3. AWS Lambda 에서 지원하지 않는 런타임 사용 가능했다.
4. 기존 Lambda 환경과 달리 완전히 캡슐화된 Lambda 함수를 제공할 수 있었다.

<br>

**단점**

- 용량 한계 극복한 점 외에 실행 시간, 실행 횟수 제한 동일했다.
  - 여전히 람다 이므로, 동일한 제약사항이 있었다.
- 이미지 생성 시간이 필요해지면서 배포 시간 증가했다.
- serverless framework 로 배포 시 일부 plugin 들과 충돌 문제가 있었다.
- Lambda Cold Start 시간 증가했다.

<br>

**추천 환경**

- 컨테이너 방식으로 개발된 환경에서 Lambda로도 지원해야할 때
- 용량이 큰 라이브러리 사용 및 용량 제한 없이 배포하고 있을 때
  - lambda 컨테이너의 최대 장점인 것 같다.
- Lambda에서의 코드가 실행되는 전체 환경에 대한 제어가 필요할 때

> \* 기존 lambda 환경에서 불편함이 없다면, lambda container로 변경해야할 이유는 없을 것 같다.

<br><br>

## Serverless 에서의 성능 향상을 위해 사용한 3가지 방법

### 개선 1. 동기 -> 비동기

![](../../images/[컨테이너]%20AWS%20Lambda%20Container를%20활용한%20서버리스%20아키텍처%20배포%20및%20성능%20향상_19.png)

![](../../images/[컨테이너]%20AWS%20Lambda%20Container를%20활용한%20서버리스%20아키텍처%20배포%20및%20성능%20향상_51.png)

> 왼쪽 동기 방식은 안전하고 확실하지만 요청이 많은 경우 힘들다.

<br>

### 개선 2. S3 Multi-Part Upload / CLoud Storage Parted Downalod

큰 용량의 비디오 데이터를 처리하던 도중에 발견하여 사용하게 되었다.

![](../../images/[컨테이너]%20AWS%20Lambda%20Container를%20활용한%20서버리스%20아키텍처%20배포%20및%20성능%20향상_06.png)

![](../../images/[컨테이너]%20AWS%20Lambda%20Container를%20활용한%20서버리스%20아키텍처%20배포%20및%20성능%20향상_13.png)

<br>

### 개선 3. Step function Parallel
![](../../images/[컨테이너]%20AWS%20Lambda%20Container를%20활용한%20서버리스%20아키텍처%20배포%20및%20성능%20향상_12.png)

<br>

## 결론

![](../../images/[컨테이너]%20AWS%20Lambda%20Container를%20활용한%20서버리스%20아키텍처%20배포%20및%20성능%20향상_16.png)