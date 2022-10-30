
> **`SysMemoryUtilization` 값이 90%를 초과하는 것은 이슈가 있는 것은 아니고, `FreeStorageSpace`, `JVMMemoryPressure`, `CPUUilization` 값을 확인해보아야 한다.**

5xx 에러가 발생했을 경우 아래 부분을 의심해볼 수 있다. 

<br>

## 확인 사항 1. 스토리지 가용 공간 확인

아래에 해당되는 경우 (1) 인스턴스 타입을 변경하거나 (2) EBS 볼륨을 늘리거나 (3) 인스턴스를 추가할 수 있다.

1. 가용 공간이 노드 스토리지의 20% or 20GB 미만인지 확인한다.
2. ISM 통해 스냅샷 저장 후 불필요한 데이터는 삭제한다.

```
Lack of available storage space
If one or more nodes in your cluster has less than 20% of available storage space, or less than 20 GB of storage space, basic write operations like adding documents and creating indexes can start to fail. Calculating storage requirements provides a summary of how OpenSearch Service uses disk space.

To avoid issues, monitor the FreeStorageSpace metric in the OpenSearch Service console and create CloudWatch alarms to trigger when FreeStorageSpace drops below a certain threshold. GET /_cat/allocation?v also provides a useful summary of shard allocation and disk usage. To resolve issues associated with a lack of storage space, scale your OpenSearch Service domain to use larger instance types, more instances, or more EBS-based storage.
```

<br>

## 확인 사항 2. JVMMemoryPressure

아래에 해당되는 경우 (1) 인스턴스 타입을 변경하거나 (2) 인스턴스를 추가할 수 있다.

마스터, 데이터 노드의 JVMMemoryPressure 수치가 95% 이상인지 확인한다. (= 95% 이상인 경우 OOM 에러가 발생할 수 있다.)


> |확인 사항|설명|
> |-|-|
> |JVMMemoryPressure maximum is >= 95% for 1 minute, 3 consecutive times <br><br>OldGenJVMMemoryPressure maximum is >= 80% for 1 minute, 3 consecutive times|The cluster could encounter out of memory errors if usage increases. Consider scaling vertically. OpenSearch Service uses half of an instance's RAM for the Java heap, up to a heap size of 32 GiB. You can scale instances vertically up to 64 GiB of RAM, at which point you can scale horizontally by adding instances.|
> |MasterJVMMemoryPressure maximum is >= 95% for 1 minute, 3 consecutive times<br><br>MasterOldGenJVMMemoryPressure maximum is >= 80% for 1 minute, 3 consecutive times|Consider using larger instance types for your dedicated master nodes. Because of their role in cluster stability and blue/green deployments, dedicated master nodes should have lower CPU usage than data nodes.|

<br>

## 확인 사항 3. CPUUtilization

아래에 해당되는 경우 (1) 인스턴스 타입을 변경하거나 (2) 인스턴스를 추가할 수 있다.

1. 마스터 노드의 CPUUtilization 수치가 지속적으로 50% 이상인지 확인합니다.
2. 데이터 노드의 CPUUtilization 수치가 지속적으로 80% 이상인지 확인합니다.

> |확인 사항|설명|
> |-|-|
> |CPUUtilization or WarmCPUUtilization maximum is >= 80% for 15 minutes, 3 consecutive times	100%|CPU utilization might occur sometimes, but sustained high usage is problematic. Consider using larger instance types or adding instances.|
>|MasterCPUUtilization maximum is >= 50% for 15 minutes, 3 consecutive times|	Consider using larger instance types for your dedicated master nodes. Because of their role in cluster stability and blue/green deployments, dedicated master nodes should have lower CPU usage than data nodes.|

<br>

### 참고

1. [Why is the SysMemoryUtilization so high on my Amazon OpenSearch Service cluster?](https://aws.amazon.com/ko/premiumsupport/knowledge-center/opensearch-high-sysmemoryutilization/?nc1=h_ls)
2. [Recommended CloudWatch alarms for Amazon OpenSearch Service](https://docs.aws.amazon.com/opensearch-service/latest/developerguide/cloudwatch-alarms.html)