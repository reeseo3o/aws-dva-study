# AWS 클라우드 개발 키트(CDK) 요약

## 1. AWS CDK란 무엇인가?

AWS CDK(Cloud Development Kit)는 JavaScript, TypeScript, Python, Java, .NET 등 익숙한 프로그래밍 언어를 사용하여 클라우드 인프라를 코드로 정의하고 프로비저닝할 수 있는 오픈 소스 소프트웨어 개발 프레임워크입니다.

## 2. CDK와 CloudFormation 비교

-   **CDK는 CloudFormation을 대체하는 것이 아니라 활용하는 방식입니다.**
-   프로그래밍 언어를 사용하여 인프라를 정의하므로, 컴파일 시 오류를 미리 발견할 수 있고, 타입 안정성을 높일 수 있습니다.
-   작성된 CDK 코드는 CloudFormation 템플릿(JSON 또는 YAML)으로 컴파일(합성)되어 AWS에 배포됩니다.
-   인프라와 애플리케이션 런타임 코드를 함께 배포할 수 있는 유연성을 제공합니다.

## 3. CDK와 SAM (Serverless Application Model) 비교

| 구분 | SAM (Serverless Application Model)                     | CDK (Cloud Development Kit)                                  |
| :--- | :----------------------------------------------------- | :----------------------------------------------------------- |
| 초점 | 서버리스 애플리케이션 (주로 Lambda)                        | 모든 AWS 서비스 지원 (CloudFormation의 상위 집합)              |
| 작성 | JSON/YAML을 사용한 선언적 템플릿                          | TypeScript, Python 등 익숙한 프로그래밍 언어 사용             |
| 활용 | 백엔드에서 CloudFormation 사용                           | 백엔드에서 CloudFormation 사용 (템플릿 생성)                  |
| 장점 | Lambda를 사용한 빠른 시작에 용이                         | 높은 유연성, 프로그래밍 언어의 장점 활용, 광범위한 서비스 지원 |

**CDK와 SAM 결합:** `cdk synth`로 CloudFormation 템플릿을 생성한 후, `sam local invoke`를 사용하여 로컬에서 CDK 앱의 Lambda 함수를 테스트할 수 있습니다.

## 4. CDK 주요 컨스트럭트 (Constructs)

CDK 컨스트럭트는 최종 CloudFormation 스택을 생성하는 데 필요한 모든 것을 담은 구성 요소입니다.

-   **계층 1 (L1 - CFN Resources):** CloudFormation 리소스와 직접적으로 매핑됩니다. `Cfn` 접두사를 가지며 (예: `s3.CfnBucket`), 모든 속성을 세세하게 제어해야 합니다. CloudFormation에서 CDK로 마이그레이션할 때 유용합니다.
-   **계층 2 (L2 - Higher-Level AWS Resources):** 더 높은 수준의 AWS 리소스를 나타냅니다 (예: `s3.Bucket`). 편리한 기본값과 보일러플레이트 코드가 줄어들어 사용이 간편하며, 추가적인 헬퍼 메서드를 제공합니다 (예: `bucket.addLifeCycleRule(...)`).
-   **계층 3 (L3 - Patterns):** 여러 관련 리소스를 묶어 흔한 작업을 수행하는 패턴을 제공합니다 (예: `ecs_patterns.ApplicationLoadBalancedFargateService`). 복잡한 아키텍처를 몇 줄의 코드로 쉽게 구성할 수 있게 해줍니다.

컨스트럭트는 CDK에 포함된 **Construct Library**나 AWS, 서드파티, 오픈 소스 커뮤니티의 컨스트럭트를 모아놓은 **Construct Hub**를 통해 얻을 수 있습니다.

## 5. 주요 CDK 명령어

-   `npm install -g aws-cdk` (또는 `npm install aws-cdk-lib`): CDK CLI 및 라이브러리 설치.
-   `cdk init app --language <언어>`: 지정된 언어로 CDK 앱 초기화 (예: `javascript`, `python`).
-   `cdk synth`: CDK 코드를 CloudFormation 템플릿으로 합성하고 출력.
-   `cdk bootstrap`: CDK 앱 배포를 위해 AWS 환경(계정 및 리전 조합)에 필요한 리소스 (S3 버킷, IAM 역할 등 포함된 `CDKToolkit` 스택)를 프로비저닝. 각 환경당 한 번만 수행.
    -   명령 형식: `cdk bootstrap aws://<YOUR_AWS_ACCOUNT_ID>/<AWS_REGION>`
-   `cdk deploy`: 스택을 AWS에 배포.
-   `cdk diff`: 로컬 CDK 코드와 배포된 CloudFormation 스택 간의 차이점 확인.
-   `cdk destroy`: 배포된 스택 삭제.

## 6. CDK 애플리케이션 실습 흐름 (예시: S3, Lambda, Rekognition, DynamoDB 연동)

1.  **CDK 초기화 및 설치:** `npm install -g aws-cdk`
2.  **CDK 앱 생성 및 초기화:** `mkdir cdk-app && cd cdk-app`, `cdk init app --language javascript`
3.  **스택 로직 수정 (`lib/cdk-app-stack.js`):**
    -   필요한 CDK 라이브러리 및 서비스 모듈 (`s3`, `iam`, `lambda`, `dynamodb` 등)을 `require`.
    -   S3 버킷, IAM 역할, Lambda 함수, DynamoDB 테이블 등의 리소스를 코드로 정의.
    -   리소스 간의 관계 및 권한 설정 (예: Lambda 함수에 S3 버킷 읽기/쓰기 권한 부여).
4.  **Lambda 함수 코드 작성 (`lambda/index.py`):**
    -   S3 이벤트 발생 시 실행될 로직 구현 (예: Rekognition을 사용한 이미지 분석, 결과를 DynamoDB에 저장).
5.  **CDK 부트스트랩:** `cdk bootstrap` (해당 계정/리전에 처음 배포 시)
6.  **CloudFormation 템플릿 합성:** `cdk synth` (생성될 템플릿 미리보기)
7.  **스택 배포:** `cdk deploy`
8.  **리소스 정리:** S3 버킷 내용물 삭제 후 `cdk destroy`

## 7. CDK 테스트

CDK 코드는 일반 프로그래밍 코드처럼 테스트할 수 있습니다. CDK 앱에는 Jest(JavaScript)나 Pytest(Python) 같은 테스트 프레임워크와 함께 사용할 수 있는 **CDK Assertion Module**이 포함되어 있습니다.

-   **테스트 유형:**
    1.  **세분화된 어서션 (Fine-grained Assertions):** 가장 일반적인 방식으로, 특정 리소스가 특정 속성을 가지고 있는지 테스트합니다. (예: Lambda 함수 런타임, SNS 토픽 구독 수).
    2.  **스냅샷 테스트 (Snapshot Tests):** 생성된 CloudFormation 템플릿을 이전에 저장된 기준 템플릿과 비교합니다.

-   **템플릿 로드 방법:**
    1.  `Template.fromStack(MyStack)`: CDK에서 정의한 스택으로부터 템플릿을 가져옵니다.
    2.  `Template.fromString(MyString)`: 문자열(파일 등)으로부터 외부 CloudFormation 템플릿을 가져와 테스트합니다.

## 8. CDK와 SAM 통합 테스트 가이드

CDK로 정의된 Lambda 함수를 로컬에서 테스트하기 위해 SAM CLI를 활용할 수 있습니다. 이를 통해 실제 AWS 환경에 배포하지 않고도 Lambda 함수의 동작을 검증할 수 있습니다.

### 8.1 사전 준비사항

1. AWS SAM CLI 설치
   ```bash
   pip install aws-sam-cli
   ```

2. Docker 설치 및 실행
   - SAM CLI는 로컬 테스트를 위해 Docker를 사용합니다.
   - Lambda 실행 환경을 에뮬레이션하기 위함입니다.

### 8.2 테스트 단계

1. CDK 코드를 CloudFormation 템플릿으로 변환
   ```bash
   cdk synth > template.yaml
   ```

2. Lambda 함수 로컬 테스트
   ```bash
   sam local invoke [Lambda 함수 논리 ID] -t template.yaml --event events/test-event.json
   ```

   - `[Lambda 함수 논리 ID]`: CloudFormation 템플릿에서 확인할 수 있는 Lambda 함수의 논리적 이름
   - `-t template.yaml`: `cdk synth`로 생성된 CloudFormation 템플릿 파일
   - `--event events/test-event.json`: 테스트에 사용할 이벤트 데이터 파일

### 8.3 고급 테스트 기능

1. 디버깅
   ```bash
   sam local invoke -d 5858 [Lambda 함수 논리 ID] -t template.yaml
   ```
   - `-d 5858`: 디버그 포트 지정

2. 환경 변수 설정
   ```bash
   sam local invoke [Lambda 함수 논리 ID] -t template.yaml --env-vars env.json
   ```

3. API Gateway 통합 테스트
   ```bash
   sam local start-api -t template.yaml
   ```

### 8.4 모범 사례

1. **이벤트 데이터 관리**
   - `events/` 디렉토리에 테스트 이벤트 JSON 파일들을 구조화하여 저장
   - 다양한 시나리오에 대한 테스트 이벤트 준비

2. **로컬 테스트 자동화**
   - 쉘 스크립트나 Makefile을 사용하여 테스트 과정 자동화
   - CI/CD 파이프라인에 통합

3. **환경 변수 관리**
   - 로컬 테스트용 환경 변수는 별도의 파일로 관리
   - 민감한 정보는 AWS Secrets Manager나 Parameter Store 사용

4. **성능 최적화**
   - 첫 실행 시 Docker 이미지 다운로드로 인한 지연 예상
   - 자주 사용하는 런타임의 이미지는 미리 다운로드

### 8.5 주의사항

1. **AWS 서비스 통합**
   - 로컬 테스트 시 실제 AWS 서비스와의 통합이 필요한 경우 적절한 IAM 권한 필요
   - 모킹(Mocking)을 통한 테스트도 고려

2. **리소스 제한**
   - 로컬 환경의 메모리와 CPU 제한 고려
   - Lambda 함수의 타임아웃 설정 확인

3. **네트워크 요구사항**
   - VPC 설정이 있는 Lambda 함수의 경우 추가 설정 필요
   - 외부 서비스 의존성 고려

## 9. AWS CDK의 추가적인 장점과 특징

### 9.1. 프로그래밍 언어의 장점 활용
- **타입 안전성**: TypeScript, Java 등의 정적 타입 언어를 사용하면 컴파일 시점에 오류를 발견할 수 있습니다.
- **IDE 지원**: 코드 자동 완성, 실시간 오류 검사, 리팩토링 도구 등 IDE의 강력한 기능을 활용할 수 있습니다.
- **재사용성**: 객체 지향 프로그래밍의 상속, 캡슐화 등을 활용하여 인프라 코드를 효율적으로 관리할 수 있습니다.
- **단위 테스트**: 일반 애플리케이션 코드처럼 인프라 코드에 대한 단위 테스트를 작성할 수 있습니다.

### 9.2. 고급 패턴 및 모범 사례
- **Construct 라이브러리**: AWS에서 제공하는 검증된 고수준 컴포넌트를 활용하여 복잡한 아키텍처를 쉽게 구현할 수 있습니다.
- **환경 분리**: 개발, 스테이징, 프로덕션 환경별로 다른 설정을 코드로 관리할 수 있습니다.
- **Infrastructure as Code (IaC)**: 버전 관리, 코드 리뷰, CI/CD 파이프라인 등 소프트웨어 개발의 모범 사례를 인프라 관리에도 적용할 수 있습니다.

### 9.3. AWS 서비스와의 통합
- **CloudFormation 기반**: CDK는 내부적으로 CloudFormation을 사용하므로, AWS의 안정적인 프로비저닝 엔진의 장점을 그대로 활용할 수 있습니다.
- **AWS 서비스 통합**: Lambda 함수, ECS 서비스, API Gateway 등 다양한 AWS 서비스를 코드로 쉽게 통합할 수 있습니다.
- **자동 롤백**: 배포 실패 시 CloudFormation의 자동 롤백 기능을 통해 안전하게 이전 상태로 복원됩니다.

### 9.4. 개발 생산성 향상
- **빠른 프로토타이핑**: 고수준 구성 요소를 사용하여 복잡한 인프라를 빠르게 구축할 수 있습니다.
- **코드 생성**: `cdk init` 명령을 통해 프로젝트 기본 구조와 필요한 설정 파일들을 자동으로 생성할 수 있습니다.
- **실시간 피드백**: `cdk diff` 명령으로 변경 사항을 배포 전에 미리 확인할 수 있습니다.
- **문서화**: 코드 자체가 문서화 역할을 하며, 주석과 README를 통해 인프라 구성을 명확하게 설명할 수 있습니다.

### 9.5. 보안 및 규정 준수
- **정책 검사**: CDK는 AWS IAM 정책, 보안 그룹 규칙 등을 코드로 정의할 때 잘못된 구성을 미리 감지할 수 있습니다.
- **감사 용이성**: 인프라 변경 사항이 코드로 관리되므로 누가, 언제, 무엇을 변경했는지 추적이 용이합니다.
- **규정 준수**: AWS Organizations의 Service Control Policy (SCP)와 통합하여 조직의 정책을 준수하도록 할 수 있습니다.