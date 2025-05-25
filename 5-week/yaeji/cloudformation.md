## AWS CloudFormation 요약: 코드형 인프라(IaC) 관리

AWS CloudFormation은 코드를 사용하여 AWS 인프라 리소스를 효과적으로 모델링하고 프로비저닝하는 강력한 서비스입니다.

### 1. CloudFormation 이란?

텍스트 파일(템플릿)에 필요한 모든 AWS 리소스(예: EC2 인스턴스, S3 버킷, VPC)를 선언적으로 정의하면, CloudFormation이 해당 리소스를 **정확한 순서**로 **예측 가능**하게 생성하고 구성합니다. 이를 **코드형 인프라(Infrastructure as Code, IaC)** 라고 부르며, 인프라를 애플리케이션 코드처럼 관리할 수 있게 해줍니다.

### 2. CloudFormation 사용의 핵심 이점

*   **자동화 및 표준화**: 수동 구성으로 인한 오류를 줄이고, 일관된 인프라 환경을 보장하며, Git 등을 통한 **버전 관리**가 가능합니다.
*   **생산성 향상**: 복잡한 인프라도 템플릿 재사용을 통해 빠르고 반복적으로 배포/업데이트 가능합니다. 개발/테스트 환경처럼 필요에 따라 쉽게 생성하고 삭제할 수 있습니다.
*   **변경 관리 및 제어**: 모든 인프라 변경이 코드를 통해 이루어지므로, 변경 사항 검토 및 추적이 용이합니다.
*   **비용 관리**: 스택별 리소스 비용 추적 및 템플릿 기반 비용 예측이 가능하며, 자동화를 통해 불필요한 리소스를 정리하여 비용을 절감할 수 있습니다.
*   **관심사 분리**: 네트워크, 애플리케이션 등 논리적 단위로 스택을 분리하여 관리 복잡성을 낮출 수 있습니다.

### 3. 작동 원리

1.  **템플릿 작성 (YAML 권장)**:
    *   필요한 AWS 리소스와 속성, 관계를 **YAML** 또는 JSON 형식으로 정의합니다.
    *   YAML은 **가독성**이 뛰어나고 **주석** 사용이 가능하여 JSON보다 권장됩니다.
    *   `AWS Application Composer`와 같은 도구를 사용하면 템플릿을 시각적으로 설계하고 작성하는 데 도움이 됩니다.
2.  **템플릿 저장**: 작성된 템플릿을 로컬 또는 S3 버킷에 저장합니다.
3.  **스택 생성/업데이트**: AWS 관리 콘솔, CLI, SDK 또는 CI/CD 파이프라인을 통해 템플릿을 기반으로 스택을 생성하거나 업데이트합니다.
4.  **스택 관리**: 생성된 리소스 그룹(스택)을 하나의 단위로 관리하며, 스택 삭제 시 관련 리소스도 함께 삭제됩니다 (DeletionPolicy 설정에 따라 예외 가능).

### 4. 핵심 템플릿 구성 요소

*   **`Resources` (필수)**: 생성하려는 AWS 리소스(EC2, S3, RDS 등)와 해당 속성을 정의합니다. 각 리소스는 스택 내에서 고유한 논리적 ID를 가집니다.
    *   **중요**: 어떤 속성을 사용할 수 있는지는 **반드시 해당 리소스의 AWS 공식 문서**를 참조하여 확인해야 합니다.
*   **`Parameters`**: 템플릿 실행 시 사용자로부터 입력받는 동적 값입니다 (예: EC2 인스턴스 유형, DB 비밀번호).
    *   `Type` (String, Number, `AWS::EC2::KeyPair::KeyName` 등 AWS 특정 유형) 지정을 통해 기본적인 유효성 검사가 가능합니다.
    *   `AllowedValues`, `AllowedPattern` 등 제약 조건을 통해 입력값을 제한할 수 있습니다.
    *   `NoEcho` 옵션으로 비밀번호 등 민감한 정보 입력을 마스킹 처리할 수 있습니다.
*   **`Mappings`**: 미리 정의된 고정 값의 맵입니다. 환경(dev/prod)이나 리전별로 다른 값(예: AMI ID)을 조건부로 선택할 때 유용합니다. `Fn::FindInMap` 함수로 값을 조회합니다.
*   **`Outputs`**: 스택 생성 후 생성된 리소스의 정보(예: 로드 밸런서 DNS 주소, VPC ID)를 출력합니다.
    *   `Export`를 통해 다른 스택에서 `Fn::ImportValue` 함수로 값을 가져와 재사용할 수 있어 스택 간 의존성 관리에 유용합니다.
*   **`Conditions`**: 특정 조건(예: 파라미터 값 비교)에 따라 리소스 생성 여부나 속성 값을 결정합니다. (예: 운영 환경에만 특정 리소스 배포). `Fn::Equals`, `Fn::And`, `Fn::If` 등의 함수를 사용합니다.
*   **`AWSTemplateFormatVersion`, `Description`**: 템플릿 버전과 설명을 명시합니다.

### 5. 주요 내장 함수

*   **`!Ref` (또는 `Fn::Ref`)**: 파라미터 값 또는 리소스의 기본 식별자(물리적 ID)를 반환합니다.
*   **`!GetAtt` (또는 `Fn::GetAtt`)**: 리소스의 특정 속성 값을 반환합니다. (예: `!GetAtt MyEC2Instance.PublicIp`).
*   **`!FindInMap` (또는 `Fn::FindInMap`)**: `Mappings` 섹션에서 조건에 맞는 값을 찾아 반환합니다.
*   **`!ImportValue` (또는 `Fn::ImportValue`)**: 다른 스택에서 `Export`된 `Outputs` 값을 가져옵니다.
*   **`!Sub` (또는 `Fn::Sub`)**: 문자열 내의 변수(`\${VariableName}`)를 지정된 값으로 대체합니다. 변수 값은 리터럴 문자열, 파라미터, 리소스 ID, 리소스 속성 등 다양하게 사용될 수 있습니다.
*   **`!Join` (또는 `Fn::Join`)**: 배열의 요소들을 구분자로 연결하여 하나의 문자열로 만듭니다.
*   **`!Base64` (또는 `Fn::Base64`)**: 문자열을 Base64로 인코딩합니다. (주로 EC2 사용자 데이터 스크립트 전달 시 사용)

### 6. 고급 기능 및 관리 전략

*   **롤백 (Rollbacks)**: 스택 생성/업데이트 실패 시, 기본적으로 이전 안정 상태로 자동 롤백되어 변경된 리소스를 정리합니다. 디버깅 목적으로 롤백 비활성화 옵션(`--disable-rollback`)도 제공됩니다.
    *   **롤백 실패 시**: 롤백 과정 자체에서 오류가 발생하면(주로 수동 변경 등 외부 요인), 스택은 `UPDATE_ROLLBACK_FAILED` 상태가 됩니다. 이때는 **문제를 수동으로 해결**한 후 콘솔이나 CLI에서 `ContinueUpdateRollback` 작업을 실행하여 롤백을 마저 완료해야 합니다.
*   **기능 (Capabilities)**: CloudFormation이 IAM 리소스(사용자, 역할 등)를 생성/업데이트하도록 허용하려면 `CAPABILITY_IAM` 또는 `CAPABILITY_NAMED_IAM`(이름 지정 시) 권한을 명시적으로 부여해야 합니다.
    *   **매크로 및 중첩 스택**: 템플릿 재사용성 및 모듈화를 위해 매크로(Macros)나 중첩 스택(Nested Stacks) 사용 시, 템플릿 변환이 필요하므로 `CAPABILITY_AUTO_EXPAND` 권한이 필요할 수 있습니다.
*   **삭제 정책 (DeletionPolicy)**: 스택 삭제 시 리소스 보존 방법을 지정합니다.
    *   `Delete` (기본값): 리소스 삭제 (단, 비어있지 않은 S3 버킷 등 일부 리소스는 실패할 수 있음).
    *   `Retain`: 스택 삭제 시에도 해당 리소스는 AWS 계정에 보존.
    *   `Snapshot`: 리소스 삭제 전 최종 스냅샷 생성 (EBS 볼륨, RDS 등 지원).
*   **스택 정책 (StackPolicy)**: 스택 업데이트 중 특정 중요 리소스(예: 운영 DB)가 의도치 않게 업데이트/삭제되는 것을 방지하는 JSON 정책입니다. 설정 시 기본적으로 모든 업데이트가 거부되므로, 허용할 리소스와 작업은 명시적으로 `Allow` 해야 합니다.
*   **종료 방지 (TerminationProtection)**: 콘솔 또는 CLI에서 실수로 스택을 삭제하는 것을 방지합니다.
*   **사용자 지정 리소스 (Custom Resources)**: CloudFormation 미지원 리소스 프로비저닝 또는 스택 생명주기(생성/업데이트/삭제) 중 사용자 지정 로직(주로 Lambda 함수 연동) 실행 시 사용됩니다. (예: 스택 삭제 전 S3 버킷 비우기 자동화).
*   **스택세트 (StackSets)**: 단일 템플릿과 작업으로 **여러 AWS 계정과 리전**에 걸쳐 스택을 일괄적으로 생성, 업데이트, 삭제할 수 있습니다. AWS Organizations와 통합하여 조직 전체에 표준화된 인프라(예: 보안 설정, 로깅 구성)를 배포하는 데 매우 유용합니다.

### 7. 실제 적용 시나리오 예시

1.  **VPC 네트워크 구축**: `vpc-network.yaml` 템플릿으로 VPC, 서브넷, 라우팅 테이블, 게이트웨이 등을 정의합니다. 환경별 CIDR이나 서브넷 구성을 `Parameters`와 `Conditions`로 관리하고, 생성된 VPC ID와 서브넷 ID를 `Outputs`로 내보냅니다.
2.  **공통 보안 그룹 관리**: SSH 접근용 등 공통 보안 그룹을 `security-groups.yaml`로 정의하고 ID를 `Outputs`로 내보냅니다.
3.  **애플리케이션 인프라 배포**: `app-stack.yaml` 템플릿으로 EC2 인스턴스, RDS 데이터베이스, 로드 밸런서 등을 정의합니다.
    *   `Parameters`로 인스턴스 유형, DB 크기 등을 입력받습니다.
    *   `Mappings`로 리전별 AMI ID를 선택합니다.
    *   `Fn::ImportValue`로 VPC 및 보안 그룹 스택에서 내보낸 ID를 가져와 사용합니다.
    *   `!Ref`와 `!GetAtt`로 리소스 간 의존성을 설정합니다.
    *   RDS 인스턴스에 `DeletionPolicy: Snapshot`을 적용합니다.
    *   운영 환경 RDS에 `StackPolicy`를 적용하여 보호합니다.
4.  **자동화된 배포 및 관리**: CI/CD 파이프라인(예: AWS CodePipeline)과 연동하여 템플릿 변경 시 자동으로 스택을 업데이트합니다. StackSets를 사용하여 새로운 AWS 계정이나 리전에 표준 애플리케이션 환경을 신속하게 배포합니다.

### 8. AWS CloudFormation StackSets

StackSets는 단일 작업으로 여러 AWS 계정 및 리전에 걸쳐 스택을 생성, 업데이트 또는 삭제할 수 있게 해주는 CloudFormation의 확장 기능입니다.

#### 주요 특징과 이점

1. **중앙 집중식 관리**
   - 관리자 계정에서 단일 지점으로 여러 계정과 리전의 스택을 관리
   - 일관된 리소스 배포와 구성 보장
   - 조직 전체의 규정 준수 및 보안 정책 적용 용이

2. **배포 옵션**
   - **동시 배포**: 여러 계정에 동시에 배포할 수 있는 최대 계정 수 지정
   - **리전 순서**: 배포할 리전의 순서 지정 가능
   - **실패 허용**: 작업이 중단되기 전 허용되는 실패 수 설정
   - **보존 기능**: 스택 세트 삭제 시에도 특정 스택 인스턴스 유지 가능

3. **권한 관리**
   - **관리자 계정**: 스택 세트를 생성하고 관리하는 계정
   - **대상 계정**: 스택이 실제로 배포되는 계정
   - **서비스 관리 권한**: AWS Organizations와 통합하여 자동 권한 관리
   - **자체 관리 권한**: IAM 역할을 통한 수동 권한 관리

4. **운영 모델 예시**
```yaml
# stackset-example.yaml
AWSTemplateFormtVersion: '2010-09-09'
Parameters:
  EnvironmentType:
    Type: String
    AllowedValues: 
      - dev
      - staging
      - prod
    Description: Environment type for resource tagging

Resources:
  S3Bucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub '${AWS::AccountId}-${EnvironmentType}-data'
      Tags:
        - Key: Environment
          Value: !Ref EnvironmentType

  SecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Common security group
      Tags:
        - Key: Environment
          Value: !Ref EnvironmentType
```

5. **배포 전략**
   - **점진적 배포**: 소수의 계정/리전으로 시작하여 점차 확장
   - **롤백 관리**: 자동 롤백 기능 및 실패한 스택의 수동 재시도 가능

6. **모니터링 및 관리**
   - **스택 작업 상태 추적**: OUTDATED, INOPERABLE, CURRENT 등의 상태 모니터링
   - **스택 드리프트 감지**: 수동 변경된 리소스 식별 및 일관성 유지 관리

7. **실제 사용 사례**
   - **보안 통제**: WAF 규칙 배포, CloudWatch 경보 표준화
   - **규정 준수**: 로깅 구성 자동화, 암호화 정책 적용
   - **리소스 표준화**: VPC 구성 통일, IAM 정책 관리

8. **모범 사례**
   - 템플릿 버전 관리 철저
   - 매개변수 사용으로 유연성 확보
   - 실패 허용치 신중하게 설정
   - 정기적인 드리프트 감지 수행
   - 충분한 테스트 후 배포

### 9. CloudFront Functions를 활용한 JWT 검증 예시

CloudFormation을 사용하여 CloudFront Functions로 JWT 검증을 구현하는 예시입니다:

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'CloudFront Function for JWT Validation'

Resources:
  JWTValidationFunction:
    Type: AWS::CloudFront::Function
    Properties:
      Name: jwt-validation
      AutoPublish: true
      FunctionConfig:
        Comment: 'Validates JWT tokens in Authorization header'
        Runtime: cloudfront-js-1.0
      FunctionCode: |
        function handler(event) {
          var request = event.request;
          var headers = request.headers;
          
          // Authorization 헤더 확인
          if (!headers.authorization) {
            return {
              statusCode: 401,
              statusDescription: 'Unauthorized',
              body: 'Authorization header is missing'
            };
          }
          
          try {
            // JWT 토큰 추출 (Bearer 스키마 사용 가정)
            var token = headers.authorization.value.split(' ')[1];
            
            // JWT 검증 로직
            if (!validateJWT(token)) {
              return {
                statusCode: 401,
                statusDescription: 'Unauthorized',
                body: 'Invalid JWT token'
              };
            }
            
            // 유효한 토큰인 경우 원본 요청 전달
            return request;
          } catch (error) {
            return {
              statusCode: 500,
              statusDescription: 'Internal Server Error',
              body: 'Error processing JWT token'
            };
          }
        }
        
        function validateJWT(token) {
          // JWT 검증 로직 구현
          // - 토큰 구조 확인
          // - 서명 검증
          // - 만료 시간 확인
          // - 발행자 확인
          // 등의 로직이 여기에 구현됨
          return true; // 실제 구현에서는 적절한 검증 후 반환
        }

  CloudFrontDistribution:
    Type: AWS::CloudFront::Distribution
    Properties:
      DistributionConfig:
        Enabled: true
        DefaultCacheBehavior:
          FunctionAssociations:
            - EventType: viewer-request
              FunctionARN: !GetAtt JWTValidationFunction.FunctionARN
          # 기타 캐시 동작 설정...
        # 기타 배포 설정...

Outputs:
  FunctionARN:
    Description: 'ARN of the CloudFront Function'
    Value: !GetAtt JWTValidationFunction.FunctionARN
  DistributionId:
    Description: 'ID of the CloudFront Distribution'
    Value: !Ref CloudFrontDistribution
```

이 템플릿의 주요 특징:

1. **CloudFront Function 정의**
   - `AWS::CloudFront::Function` 리소스 타입 사용
   - JavaScript 런타임으로 JWT 검증 로직 구현
   - `AutoPublish: true`로 설정하여 자동 배포

2. **CloudFront Distribution 연결**
   - Function을 Viewer Request 이벤트에 연결
   - `FunctionAssociations` 속성을 통해 연결 설정

3. **보안 고려사항**
   - 엣지에서 조기 요청 필터링
   - 백엔드 리소스 보호
   - 지연 시간 최소화

4. **운영 이점**
   - 서버리스 아키텍처
   - 글로벌 엣지 로케이션 활용
   - 높은 확장성과 가용성
