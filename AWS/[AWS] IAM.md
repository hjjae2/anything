## IAM

AWS Identity and Access Management(IAM)는 AWS 리소스에 대한 access를 안전하게 제어할 수 있는 웹서비스이다.

Entity(User, Group, Role)에 Policy를 적용해서 AWS 리소스(EC2, S3, RDS, ...) 에 대한 접근을 관리한다.

> Keyword : User, Group, Role, Policy

<br><br>

## Terms

### IAM Resources

user, group, role, policy, identity provider objects 

### IAM identities

identify(식별), group(그룹화)에 사용되는 IAM resource object
- IAM identity 에 policy(정책)을 연결(부여)할 수 있다.
- ex : users, groups, roles 

### IAM Entities

authentication(인증)을 위해 사용하는 IAM resource object
- ex : users and roles

### Principle

(AWS 리소스 사용을 위해) user(+ root user), role 를 사용하는 주체(person, application)
- ex: federated users, assumed roles, ...



### IAM Role

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

`Role A` 가 `Role B`와 연결되어 있을 때(`Role A`가 `Role B`를 필요로 할 때?) Role chaining 을 통해 연결된 Role 까지 사용할 수 있도록 할 수 있다.

다만, Role chaining 을 사용하면 credential(session) 정보는 최대 1시간으로 제한된다.

<br>

**Delegation**

두 계정(accounts)간에 신뢰(trust)를 설정하여, 접근 권한을 부여할 수 있다.

- The first is the account that owns the resource (the trusting account). 
- The second is the account that contains the users that need to access the resource (the trusted account).

Trusted/Trusting 계정은 다음 중 하나이다.

- 동일 계정
- Organization 하위의 다른 계정
- 다른 Organization 의 다른 계정

To delegate permission to access a resource, you create an IAM role in the trusting account that has two policies attached. The permissions policy grants the user of the role the needed permissions to carry out the intended tasks on the resource. The trust policy specifies which trusted account members are allowed to assume the role.

When you create a trust policy, you cannot specify a wildcard (*) as a principal. The trust policy is attached to the role in the trusting account, and is one-half of the permissions. The other half is a permissions policy attached to the user in the trusted account that allows that user to switch to, or assume the role. A user who assumes a role temporarily gives up his or her own permissions and instead takes on the permissions of the role. When the user exits, or stops using the role, the original user permissions are restored. An additional parameter called external ID helps ensure secure use of roles between accounts that are not controlled by the same organization.

<br>

**Federation**

<br>

**Federated user**

<br>

**Trust policy**

신뢰하는 보안 주체(principals) 를 정의한 [`JSON policy document`](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_grammar.html) 이다.

`A role trust policy`는 IAM role 에 연결된 필수적인 `resource-based policy` 이다.

`보안 주체(principals)`는 `사용자(users)`, `역할(roles)`, `계정(accounts)`, `서비스(services)` 중에서 지정되는 값이다.

> 좀 더 찾아볼 것

<br>

**Permissions policy**

하나의 `permissions document` 는 해당 role 이 수행할 수 있는 actions, resources access 를 정의한다.
- 즉, 접근 권한, 수행 권한을 정의하는 문서이다.

The document is written according to the rules of the [IAM policy language.](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies.html)

<br>

**Permissions boundary**

> " An advanced feature in which you use policies to limit the maximum permissions that an identity-based policy can grant to a role. "

maximum permissions 를 제한(limit)할 수 있는 고급 기능(an advanced feature)이다.

> " You cannot apply a permissions boundary to a service-linked role. "

<br>

**Principal**

actions, access resources 를 수행할 수 있는 하나의 엔티티(an entity)이다.

다음 종류 중 하나가 된다.

- AWS account root user
- IAM user
- IAM role

<br>

**Role for cross-account access**