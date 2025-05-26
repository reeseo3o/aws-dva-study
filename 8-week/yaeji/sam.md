## 1. AWS SAM 이란?

*   **정의**: 서버리스 애플리케이션을 보다 쉽게 빌드, 테스트, 배포할 수 있도록 지원하는 오픈 소스 프레임워크입니다.
*   **기능**:
    *   YAML 또는 JSON 형식의 템플릿을 사용하여 서버리스 애플리케이션 리소스(Lambda 함수, API Gateway, DynamoDB 테이블 등)를 간결하게 정의합니다.
    *   SAM 템플릿은 AWS CloudFormation 스택으로 변환되어 배포됩니다. 즉, CloudFormation의 확장 기능으로 볼 수 있습니다.
    *   CloudFormation의 모든 기능(파라미터, 매핑, 출력 등)을 지원합니다.
    *   내부적으로 AWS CodeDeploy를 사용하여 Lambda 함수 배포 시 점진적인 롤아웃(카나리, 선형)을 지원합니다.
    *   로컬 환경에서 Lambda 함수 및 API Gateway를 테스트하고 디버깅할 수 있는 기능을 제공합니다.
*   **SAM 템플릿 식별**: 템플릿 파일 상단에 `Transform: AWS::Serverless-2016-10-31` 헤더를 명시하여 SAM 템플릿임을 나타냅니다.

## 2. SAM 주요 구성 요소 및 리소스

*   **`AWS::Serverless::Function`**: Lambda 함수를 정의합니다.
    *   주요 속성: `Handler`, `Runtime`, `CodeUri`, `Events`, `Policies`, `Environment`.
*   **`AWS::Serverless::Api`**: API Gateway REST API를 정의합니다. Lambda 함수와 통합됩니다.
    *   주요 속성: `StageName`, `DefinitionBody` (Swagger/OpenAPI 사용 시).
*   **`AWS::Serverless::HttpApi`**: API Gateway HTTP API를 정의합니다. REST API보다 저렴하고 빠르지만 일부 기능이 제한됩니다.
*   **`AWS::Serverless::SimpleTable`**: DynamoDB 테이블을 간편하게 정의합니다.
    *   주요 속성: `PrimaryKey` (`Name`, `Type`), `ProvisionedThroughput`.
*   **`Globals` 섹션**: 여러 `AWS::Serverless::Function`, `AWS::Serverless::Api`, `AWS::Serverless::SimpleTable` 리소스에 공통적으로 적용될 속성을 정의하여 중복을 줄일 수 있습니다.
*   **`AWS::Serverless::Application`**: 중첩 애플리케이션을 정의할 때 사용합니다.

## 3. SAM CLI 주요 명령어

*   **`sam init`**: 새로운 SAM 프로젝트를 초기화합니다. 런타임, 애플리케이션 템플릿 등을 선택할 수 있습니다.
    *   예: `sam init --runtime python3.9 --name my-sam-app`
*   **`sam build`**: 애플리케이션 코드와 SAM 템플릿을 빌드하여 배포 가능한 아티팩트를 생성합니다.
    *   Lambda 함수 종속성을 패키징하고, `template.yaml`을 기준으로 빌드된 템플릿(`.aws-sam/build/template.yaml`)을 생성합니다.
    *   `--use-container` 옵션: Lambda 실행 환경과 유사한 Docker 컨테이너 내에서 빌드하여 호환성 문제를 줄입니다.
*   **`sam package`**: (구 버전 명령어, `sam deploy`에 통합됨) 빌드된 아티팩트를 S3 버킷에 업로드하고, S3 경로가 포함된 패키징된 템플릿 파일을 생성합니다.
    *   AWS CLI: `aws cloudformation package --template-file template.yaml --s3-bucket <버킷이름> --output-template-file packaged.yaml`
*   **`sam deploy`**: 애플리케이션을 AWS 클라우드에 배포합니다. CloudFormation 변경 세트를 생성하고 실행합니다.
    *   `--guided`: 대화형 모드로 배포 설정을 안내받습니다 (스택 이름, 리전, 파라미터, `samconfig.toml` 파일 생성 등).
    *   `--capabilities CAPABILITY_IAM` 또는 `CAPABILITY_NAMED_IAM`: IAM 리소스 생성 권한을 명시적으로 부여합니다.
    *   `--parameter-overrides`: 템플릿 파라미터를 CLI에서 직접 지정합니다.
    *   `--config-file samconfig.toml --config-env <환경이름>`: `samconfig.toml` 파일의 특정 환경 설정을 사용하여 배포합니다.
*   **`sam sync` (SAM Accelerate)**: 코드 또는 인프라 변경 사항을 빠르게 AWS에 배포합니다. 특히 코드 변경 시 CloudFormation을 우회하여 Lambda 함수를 직접 업데이트하므로 배포 속도가 매우 빠릅니다.
    *   `sam sync --watch`: 파일 변경을 감지하여 자동으로 동기화합니다.
    *   `sam sync --code`: 코드 변경 사항만 동기화합니다.
    *   `sam sync --resource-id <논리적ID>`: 특정 리소스만 동기화합니다.
*   **`sam logs`**: Lambda 함수의 CloudWatch Logs를 가져옵니다.
*   **`sam local invoke <FunctionName>`**: Lambda 함수를 로컬에서 호출합니다.
    *   `-e event.json`: 이벤트 페이로드 파일을 지정합니다.
*   **`sam local start-lambda`**: 로컬에서 Lambda 함수를 에뮬레이트하는 엔드포인트를 시작합니다. AWS SDK로 호출 가능합니다.
*   **`sam local start-api`**: 로컬에서 API Gateway를 에뮬레이트하여 HTTP 요청을 통해 Lambda 함수를 테스트합니다.
*   **`sam local generate-event`**: 다양한 AWS 서비스(S3, SNS, API Gateway 등)의 샘플 이벤트 페이로드를 생성합니다.
*   **`sam validate`**: SAM 템플릿의 유효성을 검사합니다.

## 4. SAM 개발 및 배포 흐름

1.  **애플리케이션 코드 작성**: Lambda 함수 로직을 작성합니다.
2.  **SAM 템플릿 정의 (`template.yaml`)**: 필요한 AWS 리소스(함수, API, 테이블 등)와 그 속성을 정의합니다.
3.  **로컬 빌드 (`sam build`)**: 애플리케이션을 빌드합니다.
4.  **(선택) 로컬 테스트**:
    *   `sam local invoke`로 함수 단위 테스트
    *   `sam local start-api`로 API 테스트
5.  **배포 (`sam deploy`)**:
    *   S3 버킷에 아티팩트 업로드 (자동 또는 `sam package` 후 수동)
    *   CloudFormation 스택 생성 또는 업데이트
6.  **배포된 애플리케이션 테스트 및 검증**

## 5. `template.yaml` 핵심 속성 및 예시

### Lambda 함수 (`AWS::Serverless::Function`)

```yaml
Resources:
  MyLambdaFunction:
    Type: AWS::Serverless::Function
    Properties:
      FunctionName: my-cool-function
      Handler: app.lambda_handler # src/app.py 파일의 lambda_handler 함수
      Runtime: python3.9
      CodeUri: src/ # Lambda 코드 위치
      Description: '''My first SAM Lambda function'''
      MemorySize: 256
      Timeout: 30
      Environment:
        Variables:
          MY_VARIABLE: '''some_value'''
          TABLE_NAME: !Ref MyDynamoTable # 다른 리소스 참조
          REGION_NAME: !Ref AWS::Region   # CloudFormation 의사 파라미터
      Events:
        HelloWorldApi: # API Gateway 이벤트 소스
          Type: Api
          Properties:
            Path: /hello
            Method: get
      Policies: # IAM 권한 부여
        - S3ReadPolicy:
            BucketName: my-data-bucket
        - DynamoDBCrudPolicy:
            TableName: !Ref MyDynamoTable
```

### API Gateway (`AWS::Serverless::Api`) - 명시적 정의 시

```yaml
Resources:
  MyApi:
    Type: AWS::Serverless::Api
    Properties:
      StageName: Prod
      # DefinitionBody: swagger.yaml # OpenAPI/Swagger 파일 경로
      # 또는 인라인으로 정의
```

### DynamoDB 테이블 (`AWS::Serverless::SimpleTable`)

```yaml
Resources:
  MyDynamoTable:
    Type: AWS::Serverless::SimpleTable
    Properties:
      TableName: my-app-table
      PrimaryKey:
        Name: id # 파티션 키 이름
        Type: String # 파티션 키 타입 (String, Number, Binary)
      ProvisionedThroughput:
        ReadCapacityUnits: 1
        WriteCapacityUnits: 1
      # SSESpecification: # 서버 측 암호화 설정
      #   SSEEnabled: True
```

## 6. SAM 정책 템플릿 (Policy Templates)

*   Lambda 함수에 공통적으로 필요한 IAM 권한을 미리 정의해둔 템플릿입니다.
*   `Policies` 속성 아래에 쉽게 적용할 수 있어 IAM 정책 작성을 간소화합니다.
*   **주요 예시**:
    *   `S3ReadPolicy`: 지정된 S3 버킷에 대한 읽기 권한 (`s3:GetObject`).
    *   `S3CrudPolicy`: 지정된 S3 버킷에 대한 CRUD 권한.
    *   `SQSPollerPolicy`: 지정된 SQS 대기열 폴링 권한 (`sqs:ReceiveMessage`, `sqs:DeleteMessage`, `sqs:GetQueueAttributes`).
    *   `DynamoDBCrudPolicy`: 지정된 DynamoDB 테이블에 대한 CRUD 권한.
    *   `DynamoDBReadPolicy`: 지정된 DynamoDB 테이블에 대한 읽기 권한.
    *   `AWSLambdaBasicExecutionRole`: CloudWatch Logs에 로그를 작성할 수 있는 기본 권한. (대부분의 경우 기본적으로 포함됨)

## 7. SAM과 CodeDeploy 연동 (점진적 배포)

*   Lambda 함수 배포 시 CodeDeploy를 활용하여 안전하고 점진적인 트래픽 이전(카나리, 선형)을 수행할 수 있습니다.
*   `AWS::Serverless::Function` 리소스의 `AutoPublishAlias`와 `DeploymentPreference` 속성을 사용합니다.

```yaml
Resources:
  MySafeDeployFunction:
    Type: AWS::Serverless::Function
    Properties:
      # ... (Handler, Runtime, CodeUri 등)
      AutoPublishAlias: live # '''live'''라는 이름의 별칭을 자동 생성 및 업데이트
      DeploymentPreference:
        Type: Canary10Percent5Minutes # 또는 Linear10PercentEvery1Minute 등
                                      # 10분 동안 10% 트래픽을 새 버전으로, 이후 100% 전환
        Alarms: # 배포 중 모니터링할 CloudWatch 경보 목록 (롤백 트리거)
          - !Ref MyCriticalAlarm
        Hooks: # 배포 전/후 실행할 Lambda 검증 함수
          PreTraffic: !Ref MyPreTrafficHookFunction
          PostTraffic: !Ref MyPostTrafficHookFunction
```

*   **`AutoPublishAlias`**: 새 버전의 Lambda 함수가 배포될 때마다 지정된 이름의 별칭(Alias)을 자동으로 생성하고 업데이트합니다.
*   **`DeploymentPreference`**:
    *   `Type`: 배포 전략 (예: `AllAtOnce`, `Canary10Percent5Minutes`, `Linear10PercentEvery2Minutes`).
    *   `Alarms`: 배포 중 문제가 발생하여 경보가 울리면 자동으로 롤백을 트리거합니다.
    *   `Hooks`: `PreTraffic` (새 버전으로 트래픽이 이전되기 전) 및 `PostTraffic` (트래픽 이전 완료 후)에 검증용 Lambda 함수를 실행하여 배포 안정성을 높입니다.

## 8. 다중 환경 관리 (`samconfig.toml`)

*   `samconfig.toml` 파일을 사용하여 개발(dev), 스테이징(staging), 운영(prod) 등 여러 환경에 대한 배포 설정을 관리할 수 있습니다.
*   `sam deploy --guided` 실행 시 생성하거나 수동으로 작성할 수 있습니다.

```toml
# samconfig.toml 예시

version = 0.1

[default.deploy.parameters]
stack_name = "my-sam-app-default"
s3_bucket = "my-default-deploy-bucket"
region = "us-east-1"
capabilities = "CAPABILITY_IAM"
# ... 기타 기본 파라미터

[dev.deploy.parameters]
stack_name = "my-sam-app-dev"
s3_bucket = "my-dev-deploy-bucket"
region = "ap-northeast-2"
parameter_overrides = "Environment=dev LogLevel=DEBUG"
# ... 기타 dev 환경 파라미터

[prod.deploy.parameters]
stack_name = "my-sam-app-prod"
s3_bucket = "my-prod-deploy-bucket"
region = "us-west-2"
parameter_overrides = "Environment=prod LogLevel=INFO"
# ... 기타 prod 환경 파라미터
```

*   **배포 시 환경 지정**: `sam deploy --config-env dev` 또는 `sam deploy --config-env prod`
    *   지정된 환경의 파라미터가 `default` 설정을 덮어쓰거나 추가하여 적용됩니다.

## 9. SAM의 장점

*   **간결성**: CloudFormation 템플릿보다 적은 코드로 서버리스 리소스를 정의할 수 있습니다.
*   **로컬 테스팅**: AWS 클라우드에 배포하지 않고도 로컬에서 함수와 API를 테스트할 수 있어 개발 생산성이 향상됩니다.
*   **모범 사례 내장**: 정책 템플릿, CodeDeploy 통합 등을 통해 AWS 모범 사례를 쉽게 적용할 수 있습니다.
*   **통합 개발 환경**: SAM CLI를 통해 빌드, 패키지, 배포, 로컬 테스트, 로그 확인 등 전체 개발 수명 주기를 지원합니다.
*   **확장성**: CloudFormation의 모든 기능을 활용할 수 있어 복잡한 아키텍처도 구성 가능합니다.

## 10. SAM 애플리케이션 배포 3단계

### 1️⃣ 로컬 환경에서 SAM 템플릿을 빌드

```bash
sam build
```

*   **목적**: 코드와 의존성을 컴파일하고 정리하여 배포 가능한 형태로 준비
*   **주요 기능**:
    *   Lambda 함수 코드와 의존성을 패키징
    *   `template.yaml`을 기준으로 `.aws-sam/build/template.yaml` 생성
    *   Lambda 런타임에 맞는 방식으로 코드 준비
*   **옵션**:
    *   `--use-container`: Docker 컨테이너 내에서 빌드하여 호환성 문제 해결

### 2️⃣ SAM 애플리케이션을 패키징

```bash
sam package --s3-bucket <your-bucket-name> --output-template-file packaged.yaml
```

*   **목적**: 빌드된 아티팩트를 Amazon S3에 업로드하고 배포 준비
*   **주요 기능**:
    *   빌드된 아티팩트를 지정된 S3 버킷에 업로드
    *   `template.yaml`을 `packaged.yaml`로 변환 (S3 링크 포함)

### 3️⃣ 패키지된 템플릿을 사용해 배포

```bash
sam deploy --template-file packaged.yaml --stack-name <stack-name> --capabilities CAPABILITY_IAM
```

*   **목적**: CloudFormation을 통해 스택 생성 및 리소스 배포
*   **주요 기능**:
    *   CloudFormation 스택 생성/업데이트
    *   Lambda 함수, API Gateway 등의 리소스 배포

### 배포 관련 추가 참고사항

1.  **배포 제한사항**:
    *   압축된 배포 패키지(.zip) 크기: 50MB
    *   압축 해제된 크기: 250MB
    *   더 큰 파일은 `/tmp` 공간 활용 고려

2.  **배포 옵션**:
    *   `--guided`: 대화형 모드로 배포 설정 안내
    *   `--capabilities CAPABILITY_IAM`: IAM 리소스 생성 권한 부여
    *   `--parameter-overrides`: 템플릿 파라미터 직접 지정

3.  **모범 사례**:
    *   배포 전 `sam validate`로 템플릿 유효성 검사
    *   로컬 테스트를 위해 `sam local invoke` 활용
    *   환경별 설정은 `samconfig.toml`로 관리

### 올바른 배포 단계 조합

✅ **정답**:
1.  로컬 환경에서 SAM 템플릿을 빌드합니다 - SAM CLI 사용 기본 단계
2.  배포를 위해 SAM 애플리케이션을 패키징합니다 - S3 업로드 필수 단계
3.  Amazon S3 버킷에서 SAM 템플릿을 배포합니다 - CloudFormation 기반 배포

❌ **오답**:
1.  Amazon EC2 인스턴스에서 SAM 템플릿을 빌드합니다 - SAM은 로컬 환경에서도 잘 동작하므로 EC2가 필수가 아님
2.  AWS CodePipeline에서 SAM 템플릿을 배포합니다 - CI/CD 자동화는 선택사항이며, 직접 배포와는 별개
3.  AWS SDK로 CodeDeploy 사용하여 SAM 템플릿 빌드 - SAM은 CodeDeploy를 배포 대상으로는 사용할 수 있지만, 빌드에 SDK는 불필요

## 11. SAM 템플릿의 Transform 섹션

### Transform 섹션의 중요성
* **정의**: `Transform` 섹션은 SAM 템플릿을 CloudFormation이 이해할 수 있는 형식으로 변환하는 데 필요한 핵심 요소입니다.
* **목적**: AWS SAM 구문을 사용하여 템플릿에서 리소스를 선언할 수 있게 합니다.
* **위치**: 템플릿 파일 상단에 위치해야 합니다.

### Transform 섹션 사용법
```yaml
Transform: AWS::Serverless-2016-10-31  # SAM 버전 지정

Resources:
  MyFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: index.handler
      Runtime: nodejs18.x
      CodeUri: ./src
```

### Transform 섹션의 특징
* **버전 지정**: `AWS::Serverless-2016-10-31`은 현재 SAM의 표준 버전을 나타냅니다.
* **자동 변환**: AWS::Serverless 변환은 AWS CloudFormation에서 호스팅되는 매크로로, SAM 구문으로 작성된 템플릿을 CloudFormation 호환 템플릿으로 자동 변환합니다.
* **리전 독립성**: Transform 섹션을 포함한 템플릿은 여러 AWS 리전에서 재사용할 수 있습니다.

### Transform 섹션 vs 다른 섹션들
* **Parameters**: 런타임에 전달되는 변수를 정의하는 섹션으로, SAM 버전과는 무관
* **Resources**: 실제 AWS 리소스를 정의하는 섹션으로, SAM 버전 지정과는 관련 없음
* **Mappings**: 조건부 값 매핑을 위한 섹션으로, SAM 변환과는 무관

### Transform 섹션 사용의 이점
1. **간단한 문법**: SAM의 간단한 문법을 사용하여 서버리스 리소스를 정의할 수 있습니다.
2. **자동 변환**: CloudFormation이 SAM 템플릿을 자동으로 변환하여 처리합니다.
3. **재사용성**: 여러 리전에서 동일한 템플릿을 재사용할 수 있습니다.
4. **유지보수**: 서버리스 애플리케이션의 인프라를 코드로 관리할 수 있습니다.

✅ **정답: Transform**
🔍 **이유:**
AWS SAM은 CloudFormation의 확장 기능입니다. CloudFormation 템플릿에서 **SAM 구문(AWS::Serverless::Function, AWS::Serverless::Api 등)**을 사용하려면,  
반드시 Transform 섹션에서 다음과 같이 매크로를 선언해야 합니다:

```yaml
Transform: AWS::Serverless-2016-10-31
```

이 선언이 있어야 CloudFormation이 SAM 템플릿을 인식하고, SAM 구문을 표준 CloudFormation 리소스로 **자동 변환**하여 처리할 수 있습니다.  
즉, Transform 섹션이 없다면 SAM 리소스 타입(`AWS::Serverless::Function` 등)을 알 수 없어 **배포가 실패**합니다.

> **실무 Tip:**  
> Transform 섹션이 누락된 경우, "Unrecognized resource types: [AWS::Serverless::Function]"와 같은 오류가 발생합니다.  
> SAM 템플릿을 작성할 때는 항상 파일 상단에 Transform 선언이 있는지 확인하세요.

---

❌ **오답 해설**

- **Parameters:**  
  템플릿에 외부 값을 전달할 때 사용하는 섹션입니다.  
  예를 들어, 환경별 변수나 사용자 입력값을 받을 때 활용하지만,  
  **SAM 활성화나 버전 지정과는 무관**합니다.

- **Mappings:**  
  조건에 따라 값을 선택하는 lookup 테이블 역할을 합니다.  
  예를 들어, 리전별 AMI ID를 매핑할 때 사용하지만,  
  **SAM 구문 활성화와는 관련이 없습니다.**

- **AWSTemplateFormatVersion:**  
  템플릿의 형식 버전을 명시하는 선언입니다.  
  CloudFormation 템플릿의 문법 버전을 나타낼 뿐,  
  **SAM 버전이나 매크로 활성화와는 아무 관련이 없습니다.**

---

### 💡 **정리**
- **Transform** 섹션이 있어야만 SAM 구문이 CloudFormation에서 동작합니다.
- 실무에서 가장 많이 하는 실수는 Transform 선언을 빼먹는 것!
- Parameters, Mappings, AWSTemplateFormatVersion은 각각 별도의 역할을 하며, SAM 활성화와는 직접적인 관련이 없습니다.

> **참고:**  
> 공식 문서: https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-specification-template-anatomy.html#sam-specification-template-anatomy-transform

### AWS::Serverless::Application 리소스 추가 설명

#### 1. 개념 및 목적
- **AWS::Serverless::Application** 리소스는 SAM 템플릿에서 **중첩 애플리케이션**(Nested Application)을 정의할 때 사용합니다.
- 중첩 애플리케이션이란, 별도의 SAM 애플리케이션(예: Serverless Application Repository 또는 S3에 저장된 패키지)을 상위 SAM 템플릿에 포함시켜, 재사용성과 모듈화를 극대화하는 방식입니다.

#### 2. 주요 속성
- `Location`: 중첩할 애플리케이션의 위치(예: S3 URL, Serverless Application Repository ARN 등)
- `Parameters`: 중첩 애플리케이션에 전달할 파라미터(선택)

#### 3. 사용 예시
```yaml
Resources:
  MyNestedApp:
    Type: AWS::Serverless::Application
    Properties:
      Location: arn:aws:serverlessrepo:ap-northeast-2:123456789012:applications/my-shared-component
      Parameters:
        StageName: prod
```
- 위 예시는 Serverless Application Repository에 등록된 공통 컴포넌트(예: 인증, 로깅 등)를 내 애플리케이션에 포함시키는 방법입니다.

#### 4. 장점 및 활용 시나리오
- **재사용성**: 여러 프로젝트에서 공통 기능(예: 인증, 모니터링, 알림 등)을 별도 애플리케이션으로 만들어 손쉽게 재사용할 수 있습니다.
- **모듈화**: 대규모 서버리스 아키텍처를 작은 단위로 분리하여 관리할 수 있습니다.
- **유지보수성**: 공통 모듈을 한 곳에서 업데이트하면, 이를 사용하는 모든 애플리케이션에 쉽게 반영할 수 있습니다.
- **Serverless Application Repository**: AWS 공식 및 커뮤니티가 제공하는 다양한 서버리스 솔루션을 손쉽게 가져와 사용할 수 있습니다.

#### 5. 실무 TIP 및 주의사항
- 중첩 애플리케이션의 파라미터는 상위 템플릿에서 오버라이드할 수 있습니다.
- 중첩 애플리케이션의 리소스 이름 충돌에 주의해야 하며, 논리적 ID가 겹치지 않도록 설계해야 합니다.
- 중첩 애플리케이션의 버전 관리(예: 특정 버전 ARN 사용)를 통해 안정적인 배포가 가능합니다.
- Serverless Application Repository에서 검증된 애플리케이션을 활용하면, 보안 및 품질 측면에서 이점이 있습니다.

#### 6. 공식 문서
- [AWS::Serverless::Application 공식 문서](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-resource-application.html)
- [Serverless Application Repository](https://aws.amazon.com/serverless/serverlessrepo/)

---

**정리**
- 중첩 애플리케이션을 임베드하려면 반드시 `AWS::Serverless::Application` 리소스 유형을 사용해야 합니다.
- 이 리소스를 활용하면, 서버리스 아키텍처의 재사용성, 모듈화, 유지보수성을 크게 높일 수 있습니다.

### Lambda 함수 배포 전략(Deployment Preference) 비교 및 실무 설명

#### 1. 주요 배포 구성 옵션 설명

- **CodeDeployDefault.HalfAtATime**
  - 전체 트래픽의 50%를 새 버전으로 전환, 나머지 50%는 이전 버전에 남김.
  - 1차 전환 후, 문제가 없으면 나머지 50%도 새 버전으로 전환(총 2단계).
  - 빠르지만, 한 번에 절반의 트래픽이 이동하므로 위험도가 높을 수 있음.

- **CodeDeployDefault.LambdaLinear10PercentEvery1Minute**
  - 1분마다 10%씩 트래픽을 새 버전으로 점진적으로 전환.
  - 10분 후 100% 전환 완료.
  - 점진적이지만, "최대한 빨리"라는 요구에는 다소 느릴 수 있음.

- **CodeDeployDefault.LambdaCanary10Percent5Minutes**
  - 처음 10% 트래픽만 새 버전으로 전환, 5분간 모니터링 후 문제가 없으면 나머지 90%를 한 번에 전환.
  - 빠른 전환과 최소한의 위험 분산을 동시에 달성.
  - 실무에서 "빠른 점진적 전환"이 필요할 때 가장 많이 사용.

- **CodeDeployDefault.LambdaLinear10PercentEvery2Minutes**
  - 2분마다 10%씩 트래픽을 새 버전으로 점진적으로 전환.
  - 20분 후 100% 전환 완료.
  - 가장 안전하지만, 전환 속도가 가장 느림.

#### 2. 실무 TIP 및 주의사항

- **카나리 배포(Canary)**: 소량의 트래픽만 새 버전에 보내어 문제 발생 시 전체 장애를 방지. 일정 시간 후 이상 없으면 전체 전환.
- **리니어 배포(Linear)**: 일정 비율씩 점진적으로 트래픽을 전환. 안정성은 높지만, 전환 속도가 느릴 수 있음.
- **AllAtOnce/HalfAtATime**: 빠르지만, 장애 발생 시 영향 범위가 큼.
- **배포 전략 선택 기준**: 서비스의 중요도, 장애 허용 범위, 배포 속도 요구사항에 따라 선택.
- **CloudWatch Alarm 연동**: 배포 중 이상 징후 감지 시 자동 롤백 설정 권장.

#### 4. SAM 템플릿 예시
```yaml
Resources:
  MyFunction:
    Type: AWS::Serverless::Function
    Properties:
      # ... (Handler, Runtime 등)
      AutoPublishAlias: live
      DeploymentPreference:
        Type: Canary10Percent5Minutes
        Alarms:
          - !Ref MyAlarm
```

#### 5. 공식 문서
- [Lambda 배포 구성 공식 문서](https://docs.aws.amazon.com/ko_kr/lambda/latest/dg/configuration-versions.html#configuration-versions-traffic-shifting)
- [AWS SAM DeploymentPreference](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-property-function-deploymentpreference.html)

---

**정리**
- "최대한 빨리, 하지만 한꺼번에 전환하지 않는" 배포에는 **CodeDeployDefault.LambdaCanary10Percent5Minutes**가 가장 적합합니다.
- 실무에서는 Canary, Linear, AllAtOnce 등 다양한 전략을 서비스 특성에 맞게 선택해야 하며, CloudWatch Alarm과 연동하여 안전한 배포를 권장합니다.
