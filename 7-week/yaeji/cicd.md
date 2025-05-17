1.  **CI/CD 개요**
    *   CI (Continuous Integration, 지속적 통합): 개발자들이 코드를 중앙 리포지토리에 자주 푸시하고, 자동으로 빌드 및 테스트하는 과정입니다. 이를 통해 버그를 조기에 발견하고 개발 주기를 개선합니다. (예: CodeBuild, Jenkins)
    *   CD (Continuous Delivery/Deployment, 지속적 제공/배포): 테스트를 통과한 코드를 자동으로 프로덕션 환경까지 배포하는 과정입니다. 수동 개입 없이 빠르고 안정적인 배포가 가능해집니다. (예: CodeDeploy, Elastic Beanstalk)

2.  **AWS CodeCommit**
    *   안전하고 확장 가능한 AWS 클라우드 기반의 프라이빗 Git 리포지토리 서비스입니다.
    *   VPC 내에 코드를 저장하며, IAM을 통한 접근 제어 및 KMS를 이용한 암호화를 지원하여 보안성이 높습니다.
    *   GitHub와 유사하게 Pull Request를 통한 코드 리뷰를 지원하며, CodeBuild 등 다른 CI/CD 도구와 통합됩니다.

3.  **AWS CodePipeline**
    *   CI/CD 파이프라인을 시각적으로 구성하고 자동화하는 워크플로 도구입니다.
    *   소스(CodeCommit, GitHub, S3 등), 빌드(CodeBuild, Jenkins 등), 테스트, 배포(CodeDeploy, Elastic Beanstalk, S3, ECS 등) 단계를 정의하고 오케스트레이션합니다.
    *   각 단계에서 생성되는 아티팩트는 S3 버킷에 저장되어 다음 단계로 전달됩니다.
    *   CloudWatch Events/EventBridge를 통해 파이프라인 상태 변화를 모니터링하고, 실패 시 CloudTrail로 API 호출을 감사합니다.
    *   실습 예제: GitHub에서 소스를 가져와 Elastic Beanstalk 환경에 배포하는 파이프라인 생성, 수동 승인 단계 추가, 변경 사항 커밋 시 자동 배포 확인.

4.  **AWS CodeBuild**
    *   소스 코드를 컴파일하고, 테스트를 실행하며, 배포 가능한 소프트웨어 패키지를 생성하는 완전 관리형 빌드 서비스입니다.
    *   빌드 지침은 프로젝트 루트의 `buildspec.yml` 파일에 정의됩니다. (필수 암기 사항)
    *   다양한 프로그래밍 언어(Java, Python, Node.js 등) 및 Docker 환경을 지원합니다.
    *   빌드 로그는 S3 및 CloudWatch Logs에 저장되며, 빌드 실패 시 EventBridge를 통해 알림을 받을 수 있습니다.
    *   실습 예제: GitHub 리포지토리와 연동하여 `buildspec.yml` 파일을 통해 특정 문자열(`Congratulations`)이 코드에 포함되어 있는지 테스트하는 빌드 프로젝트 생성 및 CodePipeline에 통합.

5.  **AWS CodeDeploy**
    *   EC2 인스턴스, 온프레미스 서버, Lambda 함수, ECS 서비스에 애플리케이션 배포를 자동화하는 서비스입니다.
    *   배포 방식은 `appspec.yml` 파일을 통해 정의됩니다.
    *   EC2/온프레미스 배포:
        *   CodeDeploy 에이전트를 대상 인스턴스에 설치해야 합니다.
        *   배포 유형: 현재 위치(In-place) 배포, 블루/그린 배포.
        *   배포 전략: AllAtOnce, HalfAtATime, OneAtATime, 사용자 정의.
    *   Lambda 배포: Lambda 별칭을 사용하여 트래픽을 점진적으로 이전합니다 (Linear, Canary, AllAtOnce).
    *   ECS 배포: 블루/그린 배포만 지원하며, 새 태스크 정의를 배포하고 트래픽을 이전합니다.
    *   롤백 기능: 배포 실패 시 또는 CloudWatch 경보 발생 시 자동으로 이전 버전으로 롤백할 수 있습니다.
    *   실습 예제: IAM 역할 생성, EC2 인스턴스에 CodeDeploy 에이전트 설치, S3에 애플리케이션 업로드, `appspec.yml`을 사용하여 EC2 인스턴스에 웹 애플리케이션 배포.

6.  **AWS CodeArtifact**
    *   소프트웨어 패키지를 안전하게 저장, 게시, 공유할 수 있는 완전 관리형 아티팩트 리포지토리 서비스입니다.
    *   Maven, Gradle, npm, pip, NuGet 등 일반적인 종속성 관리 도구와 통합됩니다.
    *   개발자나 CodeBuild가 CodeArtifact에서 직접 종속성을 가져올 수 있게 하여, 퍼블릭 리포지토리에 대한 의존성을 줄이고 보안을 강화합니다.
    *   도메인(리포지토리 집합)과 리포지토리 개념을 사용하며, 업스트림 리포지토리를 설정하여 외부 패키지를 프록시하고 캐시할 수 있습니다.
    *   EventBridge를 통해 패키지 변경 사항을 감지하고 CodePipeline 등을 트리거할 수 있습니다.
    *   리소스 정책을 사용하여 교차 계정 액세스를 허용할 수 있습니다.
    *   실습 예제: CodeArtifact 리포지토리 생성, CloudShell에서 `pip`를 사용하여 Python 패키지를 CodeArtifact를 통해 설치하고 패키지가 리포지토리에 저장되는 과정 확인.