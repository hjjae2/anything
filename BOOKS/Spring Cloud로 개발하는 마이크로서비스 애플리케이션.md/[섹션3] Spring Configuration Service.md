구성에 필요한 설정 정보(application.yml)를 외부(시스템)에서 관리한다.

하나의 저장소에서 구성 정보(application.yml)를 관리한다.

각 서비스를 재빌드하지 않고, 바로 적용할 수 있다.

'애플리케이션 배포 파이프라인'을 통해 DEV-UAT-PROD 환경에 맞는 구성 정보를 적용/사용할 수 있다.

Spring Cloud Config 서버와 같이 사용될 수 있는 저장소/솔루션은 아래와 같다.<br>
- Git Repository
- (Secure) Vault
- (Secure) File Storage

