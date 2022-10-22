### What is AWS CloudFormation?

> 요약하면, IaaS, IaaC 서비스

AWS resources 를 관리(모델링, 셋업 등)할 수 있는 서비스이다.

**1. CloudFormation Template 을 생성한다.**
   - 템플릿 : AWS 리소스 명시
   - Json, Yaml 포맷 지원 ([참고](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-whatis-concepts.html))

**2. CloudFormation 은 템플릿을 기반으로 리소스를 프로비저닝하고 구성한다.**

> *You create a template that describes all the AWS resources that you want (like Amazon EC2 instances or Amazon RDS DB instances), and CloudFormation takes care of provisioning and configuring those resources for you.*
> 
> *You don't need to individually create and configure AWS resources and figure out what's dependent on what* 

![](../images/[AWS]%20Cloudformation_52.png)

> 참고 : https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html

