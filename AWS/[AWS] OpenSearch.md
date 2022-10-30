## OpenSearch

ES의 라이센스가 변경되면서, AWS 커뮤니티에서 개발/관리하는 (AWS 기반의)ES 프로젝트가 생겨났다. (참고 : [히스토리](https://aws.amazon.com/ko/opensearch-service/the-elk-stack/what-is-elasticsearch/))

> *" 해당 (ES)라이선스는 오픈 소스가 아니며 사용자에게 동일한 자유를 제공하지 않습니다. 오픈 소스 커뮤니티와 고객이 계속해서 안전하고 고품질에 완전한 오픈 소스 검색과 분석 제품군을 사용할 수 있도록 **AWS는 커뮤니티 주도적이며 오픈 소스 Elasticsearch 및 Kibana의 ALv2 라이선스 갈래인 OpenSearch 프로젝트를 도입했습니다.** "*

이 AWS Elasticsearch 의 후속 서비스가 [AWS OpenSearch](https://docs.aws.amazon.com/ko_kr/opensearch-service/latest/developerguide/what-is.html) 이다.

<br><br>

### ISM (Index Statement Management, 인덱스 관리)

ISM(Index Statement Management)을 통해,
- 커스텀한 관리 정책(custom management policies)을 정의할 수 있다.
  - (이 정책을 통해) 주기적으로 실행하는 task를 자동화할 수 있다.
  - (이 정책을) 인덱스, 인덱스 패턴에 적용시킬 수 있다.

> 인덱스를 관리하기 위해 외부의 프로세스(예를 들어, 크론 스크립트?)를 만들었는데, ISM 덕분에 더 이상 이런 것들이 필요하지 않게 되는 것이다.

<br>

### Policy

하나의 정책은 하나의 default state 와 인덱스가 전환(transition)할 state 리스트를 가지고 있다.

각 state 내에서는, 수항핼 작업(actions)들과 전환(transitions) + 전환을 트리거할 조건(conditions)들을 정의할 수 있다. 

**정책(policy)을 인덱스(index)에 연결(attach)하면, ISM은 30~48분마다 action을 실행하고, condition을 체크하고, 인덱스의 state를 transition하는 Job을 생성한다.**

이 Job은 기본적으로 매 30분마다 동작하지만, +0~60%(즉 최대 48분 까지) jitter가 더해진다. 이유는 인덱스에 동시에 많은 작업 활동들이 급증하지 않도록 하기 위해서이다. (클러스터 상태가 빨간색이면 ISM 은 Job을 실행하지 않는다.)

ISM은 OpenSearch / Elasticsearch 6.8 이상의 버전에서 동작한다.

<br>

**예시**

일정 기간 후에 오래된 인덱스를 주기적으로 삭제할 수 있다. 인덱스 30일 후에 `read_only` 상태로 이동한 후, 90일 이후에 삭제하는 정책을 정의할 수 있다.

<br><br>

### 참고

- [AWS Elasticsearch](https://aws.amazon.com/ko/opensearch-service/the-elk-stack/what-is-elasticsearch/)
- [Amazon OpenSearch Service](https://docs.aws.amazon.com/ko_kr/opensearch-service/latest/developerguide/what-is.html)
- [Java High Level Rest Client](https://opensearch.org/docs/latest/clients/java-rest-high-level/#sample-code)