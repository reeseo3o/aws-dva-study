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