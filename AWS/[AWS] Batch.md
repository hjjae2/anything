## [Launch Template](https://docs.aws.amazon.com/batch/latest/userguide/launch-templates.html)

![](../images/[AWS]%20Batch_44.png)

> *" AWS Batch only updates the launch template with a new launch template version during infrastructure updates. For more information, see Updating compute environments. "*


> (Q) launch template 지정 시 해당 launch template 을 (직접적으로)사용하는 것인지 테스트 필요하다. (단순히 설정 정보를 읽어오는 정도(import)로만 사용하고, 별도 launch template 이 생성되는 건지)
> 
> (A) launch template 을 지정해도, (batch 에 의해)별도 launch template 이 생성되는 것을 확인했다.

<br>

### Launch template 버전 변경이 필요한 경우

[create-compute-environment](https://docs.aws.amazon.com/batch/latest/userguide/create-compute-environment.html) 문서에 나와있다.

![](../images/[AWS]%20Batch_43.png)

새로운 launch template 을 사용하기 위해서, 새로운 compute environment 생성이 필요하다.