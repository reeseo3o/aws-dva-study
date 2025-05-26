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
    *   수동 승인 작업:
        - 파이프라인의 특정 단계에서 수동 승인 작업을 추가하여 실행을 일시 중지할 수 있습니다.
        - 승인이 필요한 상황: 코드 검토, 변경 관리 검토, 수동 QA 테스트, 콘텐츠 검토 등
        - SNS 주제와 통합하여 승인 요청 알림을 이메일로 전송할 수 있습니다.
        - 승인 대기 시간은 최대 7일이며, 이 기간 내 승인되지 않으면 파이프라인이 실패합니다.
        - SNS 주제 명명 규칙 예시: `tutorialsdojoManualApprovalPHL-us-east-2-approval`
        - 승인 작업은 파이프라인과 동일한 AWS 리전의 SNS 주제를 사용해야 합니다.

    *   API Gateway 인증/인가와 CodePipeline 수동 승인 비교:

        | 구분 | API Gateway | CodePipeline 수동 승인 |
        |------|-------------|----------------------|
        | **목적** | - API 엔드포인트 외부 접근 제어<br>- 실시간 요청 인증/인가<br>- 다양한 클라이언트 접근 관리 | - CI/CD 파이프라인 워크플로우 제어<br>- 코드 배포 프로세스 품질 관리<br>- 내부 팀원 검토 프로세스 관리 |
        | **인증 방식** | - IAM (SigV4 서명)<br>- Cognito (토큰 기반)<br>- Lambda Authorizers (커스텀 로직)<br>- 리소스 정책 (IP/VPC 기반) | - IAM 권한 기반 승인/거부<br>- SNS 통한 알림<br>- 단순 승인/거부 결정 |
        | **시간 제한** | - 실시간 요청/응답<br>- Lambda Authorizer 결과 캐싱 가능 | - 최대 7일 승인 대기<br>- 시간 초과 시 자동 실패 |
        | **통합 방식** | - 다양한 백엔드 서비스 통합<br>- 서드파티 인증 시스템 통합 | - SNS 통합만 지원<br>- 동일 리전 SNS 주제 필수 |
        | **사용 사례** | - 외부 API 보안<br>- 마이크로서비스 아키텍처<br>- B2B/B2C API 제공 | - 코드 리뷰<br>- 변경 관리 검토<br>- QA 테스트 승인<br>- 콘텐츠 검토 |

4.  **AWS CodeBuild**
    *   소스 코드를 컴파일하고, 테스트를 실행하며, 배포 가능한 소프트웨어 패키지를 생성하는 완전 관리형 빌드 서비스입니다.
    *   빌드 지침은 프로젝트 루트의 `buildspec.yml` 파일에 정의됩니다. (필수 암기 사항)
    *   `buildspec.yml` 파일 구조 (시험 필수 암기):
        ```yaml
        version: 0.2
        phases:
          install:
            runtime-versions:
              nodejs: 18  # 런타임 버전 지정
            commands:
              - npm install  # 빌드 도구 설치
          pre_build:
            commands:
              - echo "Running pre-build commands"
              - npm test    # 테스트 실행
          build:
            commands:
              - echo "Running build commands"
              - npm run build  # 실제 빌드 수행
          post_build:
            commands:
              - echo "Build completed"
        artifacts:
          files:
            - '**/*'  # 빌드 결과물 지정
          name: build-artifact
        cache:
          paths:
            - 'node_modules/**/*'  # 캐시할 디렉토리
        ```
    *   phases (단계별 명령어 실행):
        - install: 빌드 환경 설정 및 종속성 설치
        - pre_build: 빌드 전 실행할 명령어 (예: 로그인, 테스트)
        - build: 실제 빌드 명령어 실행
        - post_build: 빌드 후 작업 (예: 결과물 압축, 알림)
    *   artifacts: 빌드 결과물 지정 및 관리
    *   cache: 후속 빌드를 위해 캐시할 파일/디렉토리 지정
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

    *   **배포 방식 요약**
        | 배포 방식         | 적용 대상             | 동작 방식 및 특징 |
        |------------------|----------------------|------------------|
        | 제자리(In-place) | EC2/온프레미스        | 각 인스턴스의 기존 애플리케이션을 중지하고, 최신 버전으로 교체 후 재시작 및 검증을 수행합니다. |
        | Canary           | Lambda/ECS           | 트래픽의 일부(예: 10%)만 먼저 새 버전으로 전환, 일정 시간 후 나머지 트래픽을 전환합니다. Canary 옵션에서 트래픽 비율과 대기 시간을 선택할 수 있습니다. |
        | 선형(Linear)     | Lambda/ECS           | 트래픽을 여러 단계에 걸쳐 동일한 비율로 점진적으로 새 버전으로 전환합니다. 각 단계 간의 시간(분)과 트래픽 비율을 선택할 수 있습니다. |
        | 한 번에 모두(AllAtOnce) | Lambda/ECS   | 모든 트래픽을 한 번에 새 버전으로 전환합니다. 즉시 전체 트래픽이 이동합니다. |

        - **제자리(In-place, EC2/온프레미스용)**:  
            배포 그룹의 각 인스턴스에서 애플리케이션을 중지 → 최신 애플리케이션 개정판 설치 → 새 버전 시작 및 검증 순서로 진행됩니다.

        - **Canary(Lambda/ECS용)**:  
            트래픽을 두 단계로 나누어 이동합니다. 첫 번째 단계에서 일부 트래픽(예: 10%)만 새 버전으로 전환하고, 미리 정의된 시간(분) 후 나머지 트래픽을 전환합니다. Canary 옵션에서 트래픽 비율과 대기 시간을 선택할 수 있습니다.

        - **선형(Linear, Lambda/ECS용)**:  
            트래픽을 여러 단계에 걸쳐 동일한 비율로 점진적으로 새 버전으로 전환합니다. 각 단계마다 동일한 시간(분) 간격을 두고 트래픽이 이동합니다. 선형 옵션에서 각 단계의 트래픽 비율과 시간 간격을 선택할 수 있습니다.

        - **한 번에 모두(AllAtOnce, Lambda/ECS용)**:  
            모든 트래픽이 한 번에 새 버전으로 전환됩니다. 즉시 전체 트래픽이 이동합니다.

    *   각 배포 방식은 서비스 유형(EC2/온프레미스, Lambda, ECS)에 따라 지원 여부가 다르므로, 시험에서 구분하여 암기 필요!


6.  **AWS CodeArtifact**
    *   소프트웨어 패키지를 안전하게 저장, 게시, 공유할 수 있는 완전 관리형 아티팩트 리포지토리 서비스입니다.
    *   Maven, Gradle, npm, pip, NuGet 등 일반적인 종속성 관리 도구와 통합됩니다.
    *   개발자나 CodeBuild가 CodeArtifact에서 직접 종속성을 가져올 수 있게 하여, 퍼블릭 리포지토리에 대한 의존성을 줄이고 보안을 강화합니다.
    *   도메인(리포지토리 집합)과 리포지토리 개념을 사용하며, 업스트림 리포지토리를 설정하여 외부 패키지를 프록시하고 캐시할 수 있습니다.
    *   EventBridge를 통해 패키지 변경 사항을 감지하고 CodePipeline 등을 트리거할 수 있습니다.
    *   리소스 정책을 사용하여 교차 계정 액세스를 허용할 수 있습니다.
    *   실습 예제: CodeArtifact 리포지토리 생성, CloudShell에서 `pip`를 사용하여 Python 패키지를 CodeArtifact를 통해 설치하고 패키지가 리포지토리에 저장되는 과정 확인.



