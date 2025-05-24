# AWS Step Functions

AWS Step Functions는 분산 애플리케이션 및 마이크로서비스의 구성 요소를 시각적 워크플로우를 사용하여 손쉽게 조정할 수 있게 해주는 서버리스 오케스트레이션 서비스입니다. 애플리케이션을 일련의 단계로 구축하고, 각 단계의 입력과 출력을 관리하며, 오류 처리 및 재시도 로직을 구현할 수 있습니다.

## 1. 기본 개념

*   **상태 머신 (State Machine)**: 워크플로우를 정의하는 핵심 요소입니다. JSON 기반의 Amazon States Language (ASL)를 사용하여 상태와 상태 간의 전환을 정의합니다.
*   **상태 (State)**: 워크플로우의 각 단계를 나타냅니다. 다양한 유형의 상태가 있으며, 각 상태는 특정 작업을 수행합니다.
*   **실행 (Execution)**: 상태 머신이 시작되어 워크플로우가 진행되는 인스턴스입니다. 각 실행은 고유 ID를 가지며, 진행 상황과 결과를 추적할 수 있습니다.
*   **태스크 (Task)**: 상태 머신 내에서 특정 작업을 수행하는 상태 유형입니다. Lambda 함수 호출, ECS 작업 실행, DynamoDB 항목 삽입/가져오기, SNS/SQS 메시지 게시 등 다양한 AWS 서비스와 통합될 수 있습니다.

## 1.1. Amazon States Language (ASL) 상세

Amazon States Language는 AWS Step Functions에서 상태 머신을 정의하기 위한 JSON 기반의 선언형 언어입니다. ASL을 사용하면 복잡한 워크플로우를 구조화된 방식으로 정의할 수 있습니다.

### 1.1.1. ASL의 주요 구성 요소

* **States**: 상태의 집합으로, 각각 하나의 작업을 수행합니다.
* **StartAt**: 어떤 상태에서 시작할지를 지정합니다.
* **Type**: 상태의 유형을 지정합니다.
* **Next**: 다음 상태로의 이동 경로를 지정합니다.
* **End**: 해당 상태에서 워크플로우가 종료되는지를 지정합니다.

### 1.1.2. 상태 유형 (State Types)

1. **Task State**

Task State는 Step Functions에서 가장 일반적으로 사용되는 상태 유형으로, 실제 작업을 수행하는 상태입니다.

### Task State의 주요 특징

1. **리소스 통합**
   - AWS Lambda 함수 실행
   - AWS 서비스 API 호출 (예: DynamoDB, SQS, SNS 등)
   - Activity 작업 수행 (외부 작업자와의 통합)

2. **입력/출력 처리**
   - InputPath: 입력 데이터 필터링
   - OutputPath: 출력 데이터 필터링
   - ResultPath: 작업 결과를 원본 입력과 병합
   - Parameters: 입력 데이터 변환 및 매핑

3. **오류 처리**
   - Retry: 실패 시 재시도 로직
   - Catch: 특정 오류 발생 시 대체 상태로 전환
   - TimeoutSeconds: 작업 시간 제한 설정

### Task State 예시 코드

```json
{
  "ProcessOrder": {
    "Type": "Task",
    "Resource": "arn:aws:lambda:REGION:ACCOUNT:function:ProcessOrderFunction",
    "InputPath": "$.orderDetails",
    "ResultPath": "$.processedOrder",
    "TimeoutSeconds": 300,
    "Retry": [
      {
        "ErrorEquals": ["ServiceException", "Lambda.ServiceException"],
        "IntervalSeconds": 2,
        "MaxAttempts": 3,
        "BackoffRate": 2.0
      }
    ],
    "Catch": [
      {
        "ErrorEquals": ["States.Timeout"],
        "Next": "HandleTimeout"
      }
    ],
    "Next": "ValidateOrder"
  }
}
```

### Task State를 사용한 장시간 실행 작업 분할 예시

```json
{
  "StartAt": "ProcessDataChunk1",
  "States": {
    "ProcessDataChunk1": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:REGION:ACCOUNT:function:ProcessChunk1",
      "Next": "ProcessDataChunk2",
      "TimeoutSeconds": 900
    },
    "ProcessDataChunk2": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:REGION:ACCOUNT:function:ProcessChunk2",
      "Next": "ProcessDataChunk3",
      "TimeoutSeconds": 900
    },
    "ProcessDataChunk3": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:REGION:ACCOUNT:function:ProcessChunk3",
      "Next": "ProcessDataChunk4",
      "TimeoutSeconds": 900
    },
    "ProcessDataChunk4": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:REGION:ACCOUNT:function:ProcessChunk4",
      "End": true,
      "TimeoutSeconds": 900
    }
  }
}
```

### Task State의 장점

1. **실행 제어**
   - 각 작업의 실행 시간을 개별적으로 관리
   - 작업 간의 데이터 흐름을 명확하게 정의
   - 실패한 작업만 재실행 가능

2. **모니터링 및 디버깅**
   - 각 작업의 상태와 진행 상황을 시각적으로 확인
   - 상세한 실행 이력 제공
   - CloudWatch와의 통합으로 모니터링 용이

3. **확장성**
   - 새로운 작업 추가가 용이
   - 작업 간의 의존성 명확하게 관리
   - 다양한 AWS 서비스와의 통합 지원

4. **오류 복구**
   - 세밀한 재시도 정책 설정 가능
   - 오류 발생 시 대체 경로 정의 가능
   - 장애 지점부터 재실행 가능

### Task State 사용 시 모범 사례

1. **작업 분할**
   - 큰 작업을 관리 가능한 크기로 분할
   - 각 작업의 실행 시간이 Lambda 제한 시간(15분) 이내가 되도록 설계
   - 작업 간 의존성을 명확히 정의

2. **오류 처리 전략**
   - 일시적인 오류에 대한 재시도 로직 구현
   - 영구적인 오류에 대한 대체 경로 정의
   - 적절한 타임아웃 값 설정

3. **데이터 흐름 최적화**
   - 필요한 데이터만 다음 상태로 전달
   - 큰 페이로드는 S3를 통해 전달
   - ResultPath를 사용하여 원본 입력 데이터 보존

2. **Choice State**
```json
{
  "CheckOrderAmount": {
    "Type": "Choice",
    "Choices": [
      {
        "Variable": "$.orderAmount",
        "NumericGreaterThan": 1000,
        "Next": "ApplyPremiumDiscount"
      },
      {
        "Variable": "$.orderAmount",
        "NumericGreaterThan": 500,
        "Next": "ApplyStandardDiscount"
      }
    ],
    "Default": "NoDiscount"
  }
}
```

3. **Parallel State**
```json
{
  "ProcessOrder": {
    "Type": "Parallel",
    "Branches": [
      {
        "StartAt": "UpdateInventory",
        "States": {
          "UpdateInventory": {
            "Type": "Task",
            "Resource": "arn:aws:lambda:REGION:ACCOUNT:function:UpdateInventoryFunction",
            "End": true
          }
        }
      },
      {
        "StartAt": "ChargeCustomer",
        "States": {
          "ChargeCustomer": {
            "Type": "Task",
            "Resource": "arn:aws:lambda:REGION:ACCOUNT:function:ChargeCustomerFunction",
            "End": true
          }
        }
      }
    ],
    "Next": "SendConfirmation"
  }
}
```

### 1.1.3. 오류 처리

ASL은 두 가지 주요 오류 처리 메커니즘을 제공합니다:

1. **Retry**: 작업 실패 시 재시도 로직을 정의
   * ErrorEquals: 재시도할 오류 유형
   * IntervalSeconds: 재시도 간격
   * MaxAttempts: 최대 재시도 횟수
   * BackoffRate: 재시도 간격 증가율

2. **Catch**: 특정 오류 발생 시 대체 경로 정의
   * ErrorEquals: 포착할 오류 유형
   * Next: 오류 발생 시 이동할 상태
   * ResultPath: 오류 정보를 저장할 경로

### 1.1.4. 주요 사용 사례

1. **주문 처리 워크플로우**
   * 주문 검증
   * 재고 확인
   * 결제 처리
   * 배송 처리
   * 알림 발송

2. **사용자 등록 프로세스**
   * 사용자 정보 검증
   * 계정 생성
   * 환영 이메일 발송
   * 초기 설정 가이드 제공

3. **데이터 처리 파이프라인**
   * 데이터 수집
   * 데이터 변환
   * 데이터 검증
   * 데이터베이스 저장
   * 결과 보고

### 1.1.5. ASL의 장점

1. **가시성**
   * JSON 형식으로 워크플로우를 명확하게 정의
   * Step Functions 콘솔에서 시각적으로 확인 가능

2. **유지보수성**
   * 로직이 코드가 아닌 구성으로 관리됨
   * 변경사항 적용이 용이

3. **오류 처리**
   * 재시도 로직을 선언적으로 정의
   * 오류 상황에 대한 대체 경로 지정 가능

4. **확장성**
   * 새로운 단계 추가가 용이
   * 기존 로직 수정 없이 새로운 기능 통합 가능

## 2. 주요 상태 유형

Step Functions는 워크플로우를 구성하기 위한 다양한 상태 유형을 제공합니다.

### 2.1. 태스크 상태 (Task State)

상태 머신에서 작업을 수행하는 데 사용됩니다.

*   **Lambda 함수 호출**: 특정 Lambda 함수를 동기 또는 비동기적으로 호출합니다.
*   **AWS 서비스 통합**:
    *   Amazon ECS 또는 Fargate 작업 실행 및 완료 대기
    *   DynamoDB에 직접 데이터 삽입 또는 조회
    *   Amazon SNS 주제에 메시지 게시
    *   Amazon SQS 대기열로 메시지 전송
    *   AWS Batch 작업 제출 및 완료 대기
    *   다른 Step Functions 워크플로우 실행
*   **액티비티 (Activity)**: EC2 인스턴스, 모바일 디바이스, 온프레미스 서버 등 Step Functions 외부의 작업자(worker)에게 작업을 할당하고 응답을 기다립니다. 작업자는 `GetActivityTask` API를 호출하여 작업을 가져오고, 작업 완료 후 `SendTaskSuccess` 또는 `SendTaskFailure` API를 통해 결과를 반환합니다.

**예시: Lambda 함수 호출 태스크 정의**
```json
{
  "Comment": "Lambda 함수를 호출하는 태스크 상태 예시",
  "Type": "Task",
  "Resource": "arn:aws:lambda:REGION:ACCOUNT_ID:function:FUNCTION_NAME",
  "Parameters": {
    "FunctionName": "MyLambdaFunction",
    "Payload": {
      "input.$": "$"
    }
  },
  "Next": "NextState",
  "TimeoutSeconds": 300,
  "Retry": [ {
      "ErrorEquals": [ "Lambda.ServiceException", "Lambda.AWSLambdaException", "Lambda.SdkClientException" ],
      "IntervalSeconds": 2,
      "MaxAttempts": 6,
      "BackoffRate": 2
    } ]
}
```

### 2.2. 흐름 제어 상태 (Flow Control States)

워크플로우의 실행 흐름을 제어합니다.

*   **선택 상태 (Choice State)**: 입력 데이터의 값에 따라 여러 브랜치 중 하나를 선택하여 실행 흐름을 분기합니다. `if-then-else` 로직과 유사합니다.
*   **성공 상태 (Succeed State)**: 워크플로우 실행을 성공적으로 종료합니다.
*   **실패 상태 (Fail State)**: 워크플로우 실행을 지정된 오류와 함께 실패로 종료합니다.
*   **통과 상태 (Pass State)**: 입력을 그대로 출력으로 전달하거나, 고정된 데이터를 주입합니다. 주로 디버깅이나 상태 머신 구조화에 사용됩니다.
*   **대기 상태 (Wait State)**: 지정된 시간(초) 동안 또는 특정 날짜/시간까지 실행을 일시 중단합니다.
*   **병렬 상태 (Parallel State)**: 여러 브랜치의 작업을 동시에 병렬로 실행하고 모든 브랜치가 완료될 때까지 기다립니다.
*   **맵 상태 (Map State)**: 입력 배열의 각 항목에 대해 지정된 일련의 단계를 반복적으로 실행합니다. 동적 병렬 처리에 유용합니다.

## 3. 워크플로우 스튜디오 (Workflow Studio)

AWS Management Console 내에서 제공되는 시각적 워크플로우 디자이너입니다.

*   드래그 앤 드롭 인터페이스를 사용하여 상태 머신을 시각적으로 설계하고 편집할 수 있습니다.
*   다양한 상태 유형과 서비스 통합을 손쉽게 구성할 수 있습니다.
*   설계된 워크플로우는 자동으로 Amazon States Language (ASL) JSON 정의로 변환되며, 반대로 JSON 코드를 수정하면 시각적 다이어그램도 업데이트됩니다.
*   상태 머신의 이름, 실행 유형(표준/익스프레스), IAM 역할, 로깅 설정 등을 구성할 수 있습니다.

## 4. 실행 및 모니터링

*   **실행 시작**: SDK, API Gateway, CloudWatch Events, Amazon EventBridge 또는 AWS 콘솔을 통해 수동으로 상태 머신 실행을 시작할 수 있습니다. 실행 시 JSON 형태의 입력 데이터를 제공할 수 있습니다.
*   **시각적 모니터링**: AWS 콘솔에서 각 실행의 진행 상황을 시각적으로 확인할 수 있습니다. 각 상태의 성공, 실패, 진행 중 여부가 색상으로 표시되며, 각 상태의 입력 및 출력 데이터를 검토할 수 있습니다.
*   **실행 이력**: 모든 상태 전환, 입력/출력 데이터, 발생한 오류 등의 상세 정보가 기록되어 디버깅 및 감사에 용이합니다. CloudWatch Logs와 통합하여 로그를 영구적으로 보관할 수 있습니다.

## 5. 오류 처리 (Error Handling)

Step Functions는 상태 머신 수준에서 강력한 오류 처리 메커니즘을 제공하여 애플리케이션 코드의 복잡성을 줄입니다.

### 5.1. 오류 발생 시나리오

*   **상태 머신 정의 오류**: `Choice` 상태에 일치하는 규칙이 없는 경우 등
*   **태스크 실패**: Lambda 함수 실행 중 예외 발생, 권한 부족 등
*   **일시적 오류**: 네트워크 문제, 서비스 제한 등

### 5.2. 오류 처리 방법

태스크 또는 병렬 상태 정의 내에서 `Retry` 및 `Catch` 필드를 사용하여 오류를 처리합니다.

#### 5.2.1. 재시도 (Retry)

특정 오류 발생 시 태스크 실행을 자동으로 재시도합니다. 여러 `Retrier`를 정의하여 오류 유형별로 다른 재시도 전략을 사용할 수 있습니다.

*   `ErrorEquals` (필수): 재시도할 오류 이름 배열. 미리 정의된 오류 코드 (예: `States.Timeout`, `States.TaskFailed`) 또는 사용자 정의 오류 이름을 사용할 수 있습니다. `States.ALL`은 모든 오류와 일치합니다.
*   `IntervalSeconds`: 재시도 간의 대기 시간(초). 기본값은 1.
*   `MaxAttempts`: 최대 재시도 횟수. 기본값은 3. 0이면 재시도하지 않습니다.
*   `BackoffRate`: 재시도 간격을 늘리는 배율. 예를 들어 2.0이면 재시도할 때마다 대기 시간이 두 배로 늘어납니다 (지수 백오프). 기본값은 2.0.

**예시: 재시도 로직**
```json
"Retry": [
  {
    "ErrorEquals": [ "CustomError", "States.TaskFailed" ],
    "IntervalSeconds": 5,
    "MaxAttempts": 3,
    "BackoffRate": 2.0
  },
  {
    "ErrorEquals": [ "States.Timeout" ],
    "IntervalSeconds": 30,
    "MaxAttempts": 2
  }
]
```

#### 5.2.2. 캐치 (Catch)

재시도 로직으로 처리되지 않거나 재시도 횟수를 초과한 오류를 포착하여 지정된 다음 상태로 실행 흐름을 전환합니다. 여러 `Catcher`를 정의할 수 있으며, 위에서 아래로 평가됩니다.

*   `ErrorEquals` (필수): 포착할 오류 이름 배열. `Retry`와 유사하게 사용합니다.
*   `Next` (필수): 오류 포착 시 전환할 다음 상태의 이름.
*   `ResultPath`: 오류 정보를 입력 데이터에 추가하여 다음 상태로 전달할 경로를 지정합니다. 예를 들어 `$.error_info`로 설정하면, 다음 상태의 입력에 `error_info` 필드로 오류 관련 정보(오류 이름, 원인 등)가 포함됩니다. 기본값은 `$`로, 입력 전체를 오류 정보로 대체합니다. `null`로 설정하면 오류 정보를 전달하지 않고 원래 입력만 전달합니다.

**예시: 캐치 로직**
```json
"Catch": [
  {
    "ErrorEquals": [ "CustomError" ],
    "Next": "CustomErrorFallbackState",
    "ResultPath": "$.error_details"
  },
  {
    "ErrorEquals": [ "States.ALL" ],
    "Next": "CatchAllFallbackState",
    "ResultPath": "$.error_details"
  }
]
```
애플리케이션 코드 내에서 오류를 처리하는 대신 상태 머신에서 선언적으로 오류 처리를 정의함으로써, 애플리케이션 로직을 단순화하고 오류 처리 전략 변경 시 코드 재배포 없이 유연하게 대응할 수 있습니다.

## 6. 외부 시스템과의 통합: `.waitForTaskToken` 및 액티비티 태스크

Step Functions는 외부 시스템이나 수동 작업과의 통합을 위해 `.waitForTaskToken` 서비스 통합 패턴과 액티비티 태스크를 제공합니다. 두 방식 모두 외부 작업이 완료될 때까지 워크플로우를 일시 중지하고, 해당 작업의 결과를 받아 워크플로우를 재개합니다.

### 6.1. `.waitForTaskToken` 서비스 통합 패턴

일부 AWS 서비스 통합(예: Amazon SQS `sendMessage`, Lambda `invoke`) 시 리소스 ARN에 `.waitForTaskToken`을 추가하여 사용할 수 있습니다.

1.  Step Functions는 해당 서비스 호출 시 태스크 토큰(Task Token)을 생성하여 서비스 요청에 포함시킵니다.
2.  워크플로우는 이 태스크 토큰이 `SendTaskSuccess` 또는 `SendTaskFailure` API 호출을 통해 반환될 때까지 일시 중지됩니다.
3.  외부 시스템 또는 프로세스는 작업을 완료한 후, 수신한 태스크 토큰과 작업 결과를 함께 사용하여 `SendTaskSuccess` 또는 `SendTaskFailure`를 호출합니다.
4.  Step Functions는 해당 태스크 토큰을 확인하고 워크플로우를 재개 또는 실패 처리합니다.

**예시: SQS와 `.waitForTaskToken` 사용**
```json
"Resource": "arn:aws:states:::sqs:sendMessage.waitForTaskToken",
"Parameters": {
  "QueueUrl": "https://sqs.us-east-1.amazonaws.com/123456789012/myQueue",
  "MessageBody": {
    "input.$": "$",
    "taskToken.$": "$$.Task.Token" // Task Token을 SQS 메시지에 포함
  }
},
// ...
```
이 패턴은 외부 서비스가 Step Functions 워크플로우에 다시 콜백할 수 있도록 하는 데 유용합니다.

### 6.2. 액티비티 태스크 (Activity Tasks)

장기 실행 작업, 수동 승인, 또는 Step Functions에서 직접 지원하지 않는 레거시 시스템과의 통합에 사용됩니다.

1.  상태 머신에 `Activity` 태스크 상태를 정의하고, 해당 액티비티의 ARN을 지정합니다.
2.  액티비티 워커(Worker) (예: EC2 인스턴스, 온프레미스 서버의 애플리케이션)는 `GetActivityTask` API를 주기적으로 호출하여 해당 ARN에 대한 작업을 폴링합니다.
3.  Step Functions가 액티비티 태스크 상태에 도달하면, 워커에게 작업 입력과 함께 태스크 토큰을 제공합니다.
4.  워커는 작업을 수행한 후, `SendTaskSuccess` 또는 `SendTaskFailure` API를 태스크 토큰 및 결과와 함께 호출합니다.
5.  Step Functions는 워크플로우를 재개합니다.

*   **TimeoutSeconds**: 작업이 실패로 간주될 때까지 대기하는 시간입니다.
*   **HeartbeatSeconds**: 워커가 `SendTaskHeartbeat` API를 주기적으로 호출하여 작업이 아직 진행 중임을 알릴 수 있는 최대 간격입니다. 이를 통해 `TimeoutSeconds`보다 긴 작업도 처리할 수 있으며, 최대 1년까지 대기 가능합니다.

## 7. 워크플로우 유형 (Workflow Types)

Step Functions는 두 가지 주요 워크플로우 유형을 제공합니다.

| 특징             | 표준 워크플로 (Standard Workflows)                     | 익스프레스 워크플로 (Express Workflows)                     |
| ---------------- | ------------------------------------------------------ | ------------------------------------------------------------ |
| **최대 실행 시간** | 1년                                                    | 5분                                                          |
| **실행 모델**    | 정확히 1회 실행 (Exactly-once)                         | 최소 1회 실행 (At-least-once) 또는 최대 1회 실행 (At-most-once) |
| **실행 속도**    | 초당 약 2,000회 상태 전환 (리전에 따라 다름)             | 초당 100,000회 이상 실행 시작 가능 (리전에 따라 다름)        |
| **실행 기록**    | 콘솔에서 시각적 추적 및 상세 이력 (최대 90일), CloudWatch Logs | CloudWatch Logs를 통해서만 추적 가능 (콘솔 시각화 없음)       |
| **요금 모델**    | 상태 전환 횟수 기준                                      | 실행 횟수, 실행 기간, 메모리 사용량 기준                       |
| **주요 사용 사례** | 장기 실행, 감사 가능한 워크플로, 멱등성이 필요한 작업 (예: 주문 처리, ETL 작업) | 고용량, 단기 실행, 이벤트 처리 워크플로 (예: IoT 데이터 수집, 스트리밍 데이터 처리, 마이크로서비스 오케스트레이션) |

### 7.1. 익스프레스 워크플로의 하위 유형

*   **비동기식 익스프레스 워크플로 (Asynchronous Express Workflows)**:
    *   실행을 시작하고 즉시 응답을 받습니다 (결과를 기다리지 않음).
    *   최소 1회 실행 모델. 오류 발생 시 자동으로 재시도될 수 있어 동일 작업이 여러 번 수행될 수 있으므로 멱등성 관리가 필요합니다.
    *   예: 즉각적인 응답이 필요 없는 메시지 전송 서비스.
*   **동기식 익스프레스 워크플로 (Synchronous Express Workflows)**:
    *   워크플로가 완료될 때까지 기다렸다가 결과를 반환받습니다.
    *   최대 1회 실행 모델. Step Functions가 자동으로 재시도하지 않으므로, 필요시 사용자 로직으로 재시도 구현.
    *   예: API Gateway를 통해 호출되어 즉각적인 응답을 받아야 하는 마이크로서비스 오케스트레이션.

---

# AWS AppSync

AWS AppSync는 안전하고 확장 가능한 GraphQL 및 Pub/Sub API를 쉽게 개발할 수 있게 해주는 관리형 서비스입니다. 이를 통해 애플리케이션은 필요한 데이터만 효율적으로 요청하고 받을 수 있으며, 실시간 데이터 동기화 및 오프라인 기능을 구현할 수 있습니다.

## 1. 주요 개념 및 기능

*   **GraphQL API**:
    *   AppSync의 핵심 기능으로, 클라이언트가 필요한 데이터의 구조를 직접 명시하여 요청할 수 있는 API 쿼리 언어입니다.
    *   단일 요청으로 여러 소스(DynamoDB, Lambda, Aurora Serverless, Elasticsearch, HTTP 엔드포인트 등)의 데이터를 가져오거나 수정할 수 있습니다.
    *   오버페칭(불필요한 데이터 수신) 및 언더페칭(추가 요청 필요) 문제를 해결합니다.
*   **실시간 데이터 (Real-time Data)**:
    *   GraphQL 구독(Subscription)을 통해 서버에서 클라이언트로 실시간 데이터 업데이트를 푸시할 수 있습니다.
    *   MQTT over WebSockets를 사용하여 연결된 클라이언트에게 변경 사항을 자동으로 브로드캐스트합니다.
    *   채팅 애플리케이션, 실시간 대시보드 등에 유용합니다.
*   **데이터 소스 (Data Sources)**:
    *   GraphQL 작업(쿼리, 뮤테이션, 구독)을 실제 데이터 저장소 또는 트리거에 연결하는 백엔드 리소스입니다.
    *   지원되는 데이터 소스:
        *   Amazon DynamoDB
        *   AWS Lambda
        *   Amazon Aurora Serverless
        *   Amazon OpenSearch Service (구 Elasticsearch Service)
        *   HTTP 엔드포인트 (기존 REST API 또는 다른 GraphQL API)
        *   없음 (로컬 리졸버 - Pub/Sub 전용)
*   **리졸버 (Resolvers)**:
    *   GraphQL 스키마의 필드와 데이터 소스 간의 연결 로직입니다.
    *   Apache Velocity Template Language (VTL)을 사용하여 요청 및 응답 매핑 템플릿을 작성합니다.
    *   데이터 변환, 권한 부여 로직, 조건부 데이터 가져오기 등을 수행합니다.
    *   파이프라인 리졸버(Pipeline Resolvers)를 사용하면 여러 함수(단일 작업 단위)를 순차적으로 실행하여 복잡한 비즈니스 로직을 구성할 수 있습니다.
*   **오프라인 데이터 동기화**:
    *   Amplify DataStore와 통합하여 모바일 및 웹 애플리케이션이 오프라인 상태에서도 데이터를 사용하고, 온라인 상태가 되면 자동으로 클라우드와 동기화할 수 있도록 지원합니다.
*   **보안 및 권한 부여**:
    *   API 키 (`API_KEY`)
    *   AWS IAM (`AWS_IAM`)
    *   OpenID Connect (`OPENID_CONNECT`) 공급자 (JWT 사용)
    *   Amazon Cognito 사용자 풀 (`AMAZON_COGNITO_USER_POOLS`)
    *   AWS Lambda 권한 부여 (사용자 정의 로직 실행)
    *   여러 권한 부여 모드를 동시에 사용할 수 있습니다.
*   **캐싱**:
    *   서버 측 캐싱을 통해 응답 시간을 줄이고 데이터 소스에 대한 요청 수를 줄일 수 있습니다 (전체 요청 캐싱 또는 리졸버별 캐싱).
*   **모니터링 및 로깅**:
    *   Amazon CloudWatch Logs와 통합하여 API 요청, 응답, 오류 등을 로깅합니다.
    *   CloudWatch Metrics를 통해 API 사용량, 성능, 오류율 등을 모니터링합니다.
*   **사용자 정의 도메인**:
    *   Amazon CloudFront를 AppSync API 앞에 배포하여 사용자 정의 도메인 이름 및 HTTPS를 설정할 수 있습니다.

## 2. AppSync 시작하기

1.  **스키마 정의**: GraphQL 스키마 (`schema.graphql`)를 작성하여 API의 데이터 타입, 쿼리, 뮤테이션, 구독을 정의합니다.
2.  **데이터 소스 연결**: 스키마의 필드를 처리할 데이터 소스(예: DynamoDB 테이블, Lambda 함수)를 생성하고 연결합니다.
3.  **리졸버 작성**: 각 필드에 대한 요청 및 응답 매핑 로직을 리졸버에 VTL로 작성합니다.
4.  **API 테스트 및 배포**: AppSync 콘솔의 쿼리 편집기를 사용하거나 클라이언트 애플리케이션을 통해 API를 테스트하고 배포합니다.

AppSync는 특히 모바일 및 웹 애플리케이션에서 유연하고 효율적인 데이터 접근 방식과 실시간 기능을 필요로 할 때 강력한 솔루션입니다. (이전 Cognito Sync를 대체하는 기능 제공)

---

# AWS Amplify

AWS Amplify는 안전하고 확장 가능한 풀스택(full-stack) 모바일 및 웹 애플리케이션을 신속하게 구축하고 배포할 수 있도록 지원하는 개발 프레임워크 및 호스팅 서비스입니다. 프론트엔드 UI 구축부터 백엔드 로직, 데이터 관리, 인증, 배포까지 애플리케이션 개발의 전체 라이프사이클을 단순화합니다.

## 1. Amplify의 주요 구성 요소

*   **Amplify CLI**: 로컬 개발 환경에서 Amplify 프로젝트를 구성하고 AWS 리소스를 프로비저닝하기 위한 명령줄 인터페이스입니다. `amplify init`, `amplify add <category>`, `amplify push` 등의 명령어를 사용합니다.
*   **Amplify Libraries**: iOS, Android, Web (JavaScript, React, Angular, Vue 등), React Native, Flutter와 같은 프론트엔드 프레임워크에서 AWS 백엔드 서비스와 쉽게 통합할 수 있도록 사전 구축된 UI 컴포넌트 및 클라이언트 라이브러리를 제공합니다.
*   **Amplify Studio**: Amplify CLI의 시각적 확장 기능으로, UI 개발, 백엔드 구성, 데이터 모델링, 콘텐츠 관리 등을 브라우저 기반 인터페이스에서 수행할 수 있습니다. Figma에서 UI 디자인을 가져와 React 코드로 변환하는 기능도 제공합니다.
*   **Amplify Hosting**: 최신 웹 애플리케이션(SPA, 정적 사이트) 및 서버 사이드 렌더링(SSR) 앱을 위한 완전 관리형 CI/CD 및 호스팅 서비스입니다. Git 기반 워크플로우를 지원하며, 코드 변경 시 자동으로 빌드 및 배포합니다.

## 2. Amplify가 제공하는 주요 기능 (Categories)

Amplify는 다양한 백엔드 기능을 모듈식 "카테고리"로 제공하여 필요에 따라 선택적으로 추가할 수 있습니다.

*   **인증 (Authentication)**: `amplify add auth`
    *   Amazon Cognito를 기반으로 사용자 가입, 로그인, 로그아웃, 비밀번호 복구, MFA(다중 인증) 등의 기능을 쉽게 추가할 수 있습니다.
    *   소셜 로그인(Facebook, Google, Amazon 등) 통합을 지원합니다.
    *   사전 구축된 UI 컴포넌트를 제공하여 인증 흐름을 빠르게 구현할 수 있습니다.
*   **API (GraphQL & REST)**: `amplify add api`
    *   **GraphQL**: AWS AppSync를 사용하여 GraphQL API를 구축하고 데이터 소스(DynamoDB 등)와 연결합니다. 데이터 모델링, 자동 스키마 생성 등을 지원합니다.
    *   **REST**: Amazon API Gateway와 AWS Lambda를 사용하여 REST API를 구축합니다.
*   **데이터스토어 (DataStore)**: `amplify add api` (GraphQL 선택 시)
    *   온라인/오프라인 데이터 동기화 기능을 제공하는 로컬 우선(local-first) 프로그래밍 모델입니다.
    *   클라이언트 측에서 데이터를 쿼리하고 조작하면, Amplify DataStore가 자동으로 AWS AppSync 및 DynamoDB와 데이터를 동기화합니다.
    *   네트워크 연결 없이도 앱이 작동하며, 연결이 복구되면 변경 사항을 자동으로 동기화합니다.
*   **스토리지 (Storage)**: `amplify add storage`
    *   Amazon S3를 사용하여 이미지, 비디오, 문서 등의 사용자 콘텐츠를 안전하게 저장하고 관리합니다.
    *   콘텐츠 접근 수준(공개, 보호, 비공개)을 쉽게 설정할 수 있습니다.
*   **함수 (Functions)**: `amplify add function`
    *   AWS Lambda를 사용하여 서버리스 백엔드 로직을 작성하고 배포합니다.
    *   API, 스토리지 트리거 등 다른 Amplify 카테고리와 쉽게 통합됩니다.
*   **호스팅 (Hosting)**: `amplify add hosting` / `amplify publish`
    *   Amplify Hosting을 통해 웹 애플리케이션을 배포합니다.
    *   Git 리포지토리(GitHub, GitLab, Bitbucket, AWS CodeCommit)와 연결하여 CI/CD 파이프라인을 설정합니다.
    *   커스텀 도메인, SSL/TLS 인증서, 기능 브랜치 배포, 암호 보호 등을 지원합니다.
*   **예측 (Predictions)**: `amplify add predictions`
    *   Amazon AI/ML 서비스를 활용하여 텍스트 번역, 텍스트 음성 변환, 이미지 인식 등의 기능을 앱에 추가합니다.
*   **상호작용 (Interactions)**: `amplify add interactions`
    *   Amazon Lex를 사용하여 챗봇 기능을 앱에 통합합니다.
*   **분석 (Analytics)**: `amplify add analytics`
    *   Amazon Pinpoint를 사용하여 사용자 행동 추적 및 분석 기능을 추가합니다.

## 3. Amplify Hosting의 CI/CD 및 테스트

*   **빌드, 배포, 호스팅**: Git 리포지토리에 코드를 푸시하면 Amplify Hosting이 자동으로 프론트엔드 및 백엔드 변경 사항을 빌드하고 배포합니다.
*   **빌드 설정**: `amplify.yml` 파일을 통해 빌드 및 테스트 단계를 사용자 정의할 수 있습니다.
*   **테스트**:
    *   **단위 테스트 (Unit Tests)**: 빌드 단계에서 실행되어 코드의 개별 단위가 예상대로 작동하는지 확인합니다.
    *   **종단간 테스트 (End-to-End, E2E Tests)**: 테스트 단계에서 실행되며, 애플리케이션 전체 흐름을 사용자 관점에서 검증합니다. Cypress와 같은 프레임워크를 사용하여 UI 상호작용을 시뮬레이션하고 회귀를 포착할 수 있습니다.
*   **기능 브랜치 배포 및 풀 리퀘스트 미리보기**: 각 Git 브랜치 또는 풀 리퀘스트에 대해 별도의 임시 환경을 자동으로 배포하여 변경 사항을 격리된 환경에서 테스트할 수 있습니다.

Amplify는 개발자가 인프라 관리보다는 애플리케이션 기능 개발에 집중할 수 있도록 도와주며, AWS의 강력한 서비스를 활용하여 확장 가능하고 안정적인 애플리케이션을 빠르게 구축할 수 있는 환경을 제공합니다. 