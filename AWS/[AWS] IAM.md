### IAM roles

하나의 `IAM role` 은 하나의 `IAM user` 와 유사하다.

- Role 에 Permission, Policy 등을 설정하고, 사용한다.
- 해당 Role 이 필요한 곳에서 사용한다.
- 사용자(users), 애플리케이션(applications), 서비스(services)에게 AWS 리소스에 대한 권한을 위임할 수 있다.

<br>

### [Roles terms and concepts](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_terms-and-concepts.html)

**Roles**

특정한 권한(specific permission)을 갖는 IAM identity

나머지는 위 내용(IAM Roles 내용)과 동일하다.

<br>

**AWS service role**

하나의 서비스(a service)가 작업(actions)을 수행하기 위해 role 을 사용할 수 있다.

AWS service environments 를 셋업할 때, 우리는 서비스(service)가 사용할 role 을 정의해야한다.
- 해당 서비스가 사용/접근할 AWS resources 에 대한 모든 권한이 정의되어있어야 한다.

<br>

**AWS service role for an EC2 instance**

AWS service role 중 EC2 서비스를 위한 특별한 service role 이다. (EC2만을 위해 명시적으로 하나 더 있다고 보면 될 것 같다.)

이 role 은 EC2 인스턴스가 시작될 때(be launched), 할당된다.

> For details about using a service role for an EC2 instance, see [Using an IAM role to grant permissions to applications running on Amazon EC2 instances.](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2.html)

<br>

**AWS service-linked role**

<br>

**Role chaining**

<br>

**Delegation**

<br>

**Federation**

<br>

**Federated user**

<br>

**Trust policy**

<br>

**Permissions policy**

하나의 `permissions document` 는 해당 role 이 수행할 수 있는 actions, resources access 를 정의한다.
- 즉, 접근 권한, 수행 권한을 정의하는 문서이다.

The document is written according to the rules of the [IAM policy language.](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies.html)

<br>

**Permissions boundary**

<br>

**Principal**

actions, access resources 를 수행할 수 있는 하나의 엔티티(an entity)이다.

다음 종류 중 하나가 된다.

- AWS account root user
- IAM user
- IAM role