> Elastic Beanstalk 에 대한 기본 개념은 어렵지 않고 [문서](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/Welcome.html)에 잘 설명이 되있어서 문서만 참고해도 좋을 듯 하다.

### 요약

1. 애플리케이션 코드만 업로드하면 서버를 운영할 수 있다.
2. 기타 인프라 설정(SG, ASG, EC2, LB, CloudWatch)은 자동으로 해준다.

### [What is AWS Elastic Beanstalk?](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/Welcome.html)

> *Elastic Beanstalk를 사용하면 애플리케이션을 실행하는 인프라에 대해 자세히 알지 못해도 AWS 클라우드에서 애플리케이션을 신속하게 배포하고 관리할 수 있습니다. Elastic Beanstalk를 사용하면 선택 또는 제어에 대한 제한 없이 관리 복잡성을 줄일 수 있습니다. 애플리케이션을 업로드하기만 하면 Elastic Beanstalk에서 용량 프로비저닝, 로드 밸런싱, 조정, 애플리케이션 상태 모니터링에 대한 세부 정보를 자동으로 처리합니다.*

> *Elastic Beanstalk를 사용하려면 애플리케이션을 생성하고, 애플리케이션 소스 번들의 형태(예: Java .war 파일)로 애플리케이션 버전을 Elastic Beanstalk에 업로드하고, 애플리케이션에 대한 몇 가지 정보를 제공합니다. Elastic Beanstalk가 자동으로 환경을 실행하고 코드 실행에 필요한 AWS 리소스를 생성 및 구성합니다. 환경 실행 후에는 환경을 직접 관리하고 새로운 앱 버전을 배포할 수 있습니다. 다음 다이어그램은 Elastic Beanstalk의 워크플로를 보여 줍니다.*
>
> [요약]
> 1. 애플리케이션 생성, 소스 번들 형태로 업로드 & 환경 설정
> 2. EB가 자동으로 환경을 구성하고 실행

![](../images/[AWS]%20ElasticBeanstalk_36.png)

> *애플리케이션을 생성 및 배포한 후에는 지표, 이벤트, 환경 상태 등의 애플리케이션 정보를 Elastic Beanstalk 콘솔, API 또는 통합된 AWS CLI를 비롯한 명령줄 인터페이스를 통해 확인할 수 있습니다.*

<br>

### 비용

Elastic Beanstalk 에 추가적인 비용은 없다. AWS Resource 에 대한 비용만 지불하면 된다.