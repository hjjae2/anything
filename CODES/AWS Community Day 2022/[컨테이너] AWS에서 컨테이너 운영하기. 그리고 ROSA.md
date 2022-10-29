![](../../images/[컨테이너]%20AWS에서%20컨테이너%20운영하기.%20그리고%20ROSA_57.png)

<br>

## PaaS

- AWS Lightsail
- AWS Elastic Beanstalk

<br><br>

## serverless

대표적으로 아래와 같은 서비스를 제공합니다.

- AWS Lambda
  - What is cold-start (issue)?
- AWS App Runner
- AWS Fargate

![](../../images/[컨테이너]%20AWS에서%20컨테이너%20운영하기.%20그리고%20ROSA_31.png)


<br><br>

## orchestration

대표적으로 아래와 같은 서비스를 제공합니다.

- AWS ECS
- AWS EKS

![](../../images/[컨테이너]%20AWS에서%20컨테이너%20운영하기.%20그리고%20ROSA_11.png)

### AWS ECS

EKS에 비해 **단순함** 을 강조한 서비스

### AWS EKS

ECS 에 비해 조금 복잡하지만 **유연성(확장성)** 을 강조한 서비스

> 입문자 입장에서는 ECS를 사용하다가 EKS를 사용하는 것을 권장한다.

<br><br>

## ROSA

**RedHat + OpenShift + AWS**

OpenShift는 k8s + 다양한 기능을 사용할 수 있는 서비스이다.

![](../../images/[컨테이너]%20AWS에서%20컨테이너%20운영하기.%20그리고%20ROSA_26.png)

### Security

클라우드 운영 시 신경써야 하는 부분이 많이 있다. 그 중 첫 번째 허들은 보안이다.

4