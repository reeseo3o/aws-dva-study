# AWS 서버리스 및 Lambda 개요

## 서버리스(Serverless)란?

- **정의**: 개발자가 서버를 직접 관리할 필요 없이 코드를 배포하고 실행할 수 있는 클라우드 컴퓨팅 모델입니다. 서버가 없는 것이 아니라, 서버 관리가 추상화된 것입니다.
- **핵심**: 코드(기능) 배포에 집중할 수 있습니다.
- **초기 의미**: FaaS (Function as a Service), 즉 서비스로서의 기능. AWS Lambda가 대표적입니다.
- **현재 의미**: FaaS를 넘어 원격으로 관리되는 모든 것 (데이터베이스, 메시징, 스토리지 등)을 포함합니다. 서버 프로비저닝이 필요 없는 서비스들을 지칭합니다.

### AWS에서의 서버리스

- **일반적인 아키텍처 예시**:
  1.  **사용자**: 웹사이트 또는 CloudFront + S3를 통해 정적 콘텐츠 접근.
  2.  **인증**: Cognito를 통해 로그인 및 사용자 신원 관리.
  3.  **API 호출**: API Gateway를 통해 REST API 실행.
  4.  **백엔드 로직**: API Gateway가 Lambda 함수를 트리거.
  5.  **데이터 처리**: Lambda 함수가 DynamoDB에서 데이터 저장 및 검색.

### AWS의 주요 서버리스 서비스

- **컴퓨팅**:
  - **AWS Lambda**: 이벤트 기반 코드 실행.
  - **AWS Fargate**: 서버 관리 없이 컨테이너 실행 (ECS/EKS).
- **데이터베이스**:
  - **Amazon DynamoDB**: NoSQL 키-값 및 문서 데이터베이스.
  - **Amazon Aurora Serverless**: 온디맨드 자동 확장 관계형 데이터베이스.
- **API**:
  - **Amazon API Gateway**: RESTful API 및 WebSocket API 생성, 관리.
- **스토리지**:
  - **Amazon S3**: 객체 스토리지.
- **메시징**:
  - **Amazon SNS (Simple Notification Service)**: 게시/구독 메시징.
  - **Amazon SQS (Simple Queue Service)**: 메시지 큐 서비스.
- **데이터 스트리밍**:
  - **Amazon Kinesis Data Firehose**: 데이터 스트림 로드 및 처리.
- **워크플로우**:
  - **AWS Step Functions**: 분산 애플리케이션 및 마이크로서비스 워크플로우 조정.
- **인증**:
  - **Amazon Cognito**: 사용자 가입, 로그인 및 액세스 제어.

**참고**: SQS, SNS, Kinesis Data Firehose 등은 서버 관리가 필요 없고 사용량에 따라 자동 확장되므로 서버리스 범주에 포함됩니다.

---

## AWS Lambda 심층 분석

Lambda는 서버를 프로비저닝하거나 관리하지 않고 코드를 실행할 수 있게 해주는 컴퓨팅 서비스입니다.

### Lambda vs. EC2

| 특징          | AWS Lambda                         | Amazon EC2                   |
| :------------ | :--------------------------------- | :--------------------------- |
| **관리 단위** | 가상 함수 (코드)                   | 가상 서버 (인스턴스)         |
| **서버 관리** | 필요 없음                          | 필요 (프로비저닝, 관리)      |
| **실행 방식** | 이벤트 기반, 온디맨드              | 지속적 실행 (시작/중지 가능) |
| **실행 시간** | 단기 실행 (최대 15분)              | 제한 없음                    |
| **확장성**    | 자동 확장                          | Auto Scaling Group 설정 필요 |
| **과금 기준** | 호출 수 + 컴퓨팅 시간 (100ms 단위) | 인스턴스 실행 시간           |

### Lambda의 장점

- **간편한 가격 구조**:
  - 호출 수와 컴퓨팅 시간(RAM 할당량 및 실행 시간 기준)에 따라 과금됩니다.
  - 매우 관대한 **무료 사용량(Free Tier)** 제공: 월 100만 건 요청 및 40만 GB-초 컴퓨팅 시간.
- **다양한 AWS 서비스와 통합**: API Gateway, S3, DynamoDB, Kinesis, SNS, SQS 등과 쉽게 연동됩니다.
- **다양한 프로그래밍 언어 지원**: Node.js(JavaScript), Python, Java, C#(.NET Core), Go, PowerShell, Ruby 등. **커스텀 런타임 API**를 통해 다른 언어(Rust 등)도 지원합니다.
- **컨테이너 이미지 지원**: Docker 컨테이너 이미지를 Lambda 함수로 배포할 수 있습니다 (Lambda Runtime API 구현 필요).
  - **시험 참고**: 일반적인 Docker 이미지 실행은 **EKS**나 **Fargate**가 더 적합할 수 있습니다. Lambda의 컨테이너 지원은 특정 사용 사례에 유용합니다.
- **쉬운 모니터링**: Amazon CloudWatch와 자동으로 통합되어 로그 및 메트릭 확인이 용이합니다.
- **유연한 리소스 할당**:
  - 함수당 최대 **10GB RAM**까지 할당 가능.
  - **RAM 증가는 CPU 및 네트워크 성능 향상**으로 이어집니다.

### Lambda와 AWS 서비스 통합 예시

Lambda는 다양한 AWS 서비스와 연동되어 이벤트 기반 아키텍처를 구축하는 데 핵심적인 역할을 합니다.

- **API Gateway**: REST API 엔드포인트를 생성하고 백엔드 Lambda 함수를 호출합니다.
- **Amazon S3**: S3 버킷에 객체 생성/삭제 등의 이벤트 발생 시 Lambda 함수를 트리거합니다.
- **Amazon DynamoDB**: 테이블의 데이터 변경(스트림) 발생 시 Lambda 함수를 트리거합니다.
- **Amazon Kinesis**: 실시간 데이터 스트림 처리 및 변환에 Lambda를 사용합니다.
- **Amazon SNS**: 특정 주제(Topic)에 메시지가 게시되면 Lambda 함수를 트리거합니다.
- **Amazon SQS**: 큐에 메시지가 도착하면 Lambda 함수를 트리거하여 처리합니다.
- **Amazon CloudWatch Events / EventBridge**: 예약된 시간(Cron) 또는 AWS 내 특정 이벤트 발생 시 Lambda 함수를 트리거합니다.
- **AWS CloudTrail**: API 호출 로그를 기반으로 Lambda 함수를 트리거합니다.
- **Amazon Cognito**: 사용자 풀 이벤트(예: 회원 가입 확인) 발생 시 Lambda 함수를 트리거합니다.
- **AWS IoT**: IoT 장치에서 메시지가 수신되면 Lambda 함수를 트리거합니다.
- **Amazon Lex**: 챗봇의 로직을 Lambda 함수로 구현합니다.
- **Amazon Alexa**: Alexa 스킬의 백엔드 로직을 Lambda 함수로 구현합니다.
- **AWS CodePipeline**: 파이프라인 상태 변경 시 Lambda 함수를 트리거하여 자동화 작업을 수행합니다.
- **CloudFront (Lambda@Edge)**: CDN 엣지 로케이션에서 코드를 실행하여 사용자에게 더 가까운 곳에서 요청/응답을 수정합니다.

### Lambda 활용 사례

1.  **서버리스 썸네일 생성**:
    - **트리거**: 사용자가 S3 버킷에 원본 이미지를 업로드합니다 (S3 이벤트 알림).
    - **처리**: Lambda 함수가 트리거되어 이미지 처리 라이브러리를 사용해 썸네일을 생성합니다.
    - **저장**: 생성된 썸네일을 다른 S3 버킷이나 동일한 S3 버킷의 다른 경로에 저장합니다.
    - **(선택)** Lambda 함수가 이미지 메타데이터(이름, 크기, 생성일 등)를 DynamoDB 테이블에 저장합니다.
    - **결과**: 이미지 업로드 시 자동으로 썸네일이 생성되는 반응형 아키텍처 구축.

2.  **서버리스 크론 작업 (Scheduled Tasks)**:
    - **기존 방식**: EC2 인스턴스에서 `cron` 데몬을 실행하여 주기적인 작업 수행 (인스턴스 유휴 시간 동안 비용 발생).
    - **서버리스 방식**:
      - **트리거**: CloudWatch Events 또는 EventBridge 규칙을 설정하여 특정 시간 간격(예: 매시간, 매일 오전 9시) 또는 특정 이벤트 발생 시 트리거되도록 합니다.
      - **처리**: 설정된 규칙이 Lambda 함수를 호출하여 필요한 작업을 수행합니다.
    - **결과**: EC2 인스턴스 없이 예약된 작업을 안정적이고 비용 효율적으로 실행.

### Lambda 가격 상세 설명

- **호출(Request) 요금**:
  - 최초 100만 건/월: 무료
  - 이후 100만 건당: $0.20 (매우 저렴)
- **지속 시간(Duration) 요금**:
  - 함수에 할당된 **메모리(GB)**와 함수 **실행 시간(초)**을 곱한 **GB-초** 단위로 계산됩니다. 실행 시간은 100ms 단위로 올림 처리됩니다.
  - 무료 사용량: 40만 GB-초/월
    - 예: 1GB RAM 함수 -> 40만 초 무료 실행 가능
    - 예: 128MB (0.125GB) RAM 함수 -> 320만 초 무료 실행 가능 (400,000 / 0.125)
    - 무료 사용량 초과 시: 지역별로 다르지만, 대략 60만 GB-초당 $1 정도입니다. (정확한 최신 정보는 AWS 요금 페이지 참조)

**결론**: Lambda는 이벤트 기반의 짧은 작업을 처리하는 데 매우 비용 효율적이며, 다양한 AWS 서비스와 쉽게 통합되어 강력한 서버리스 애플리케이션 구축을 가능하게 합니다.

---

[실습]

1.  **람다 초기화 관련 사항**:
    *   람다 함수가 처음 호출될 때 또는 한동안 호출되지 않았다가 다시 호출될 때 초기화 과정(init code)이 발생합니다.
    *   이 초기화 시간은 핸들러 함수 외부에서 수행되며, SDK 클라이언트 초기화, 데이터베이스 커넥션 설정 등을 이 단계에서 수행하여 실제 요청 처리 시간을 줄일 수 있습니다.
2.  **CloudShell을 이용한 실습 단계**:
    *   `aws lambda list-functions`: 현재 리전의 람다 함수 목록을 확인합니다.
    *   람다 함수를 인터넷에 노출시키는 방법:
        *   Application Load Balancer (ALB) 사용
        *   API Gateway 사용

### ALB와 Lambda 연동 시 고려사항

*   **HTTP에서 JSON으로 변환**: ALB가 Lambda로 요청을 전달할 때 HTTP 요청은 특정 구조의 JSON 문서로 변환됩니다.
*   **ALB Multi Header Values 지원**: (시험 출제 포인트!)
    *   ALB 설정에서 다중 헤더 값(Multi-Header Values)을 활성화할 수 있습니다.
    *   활성화 시, 동일한 키를 가진 여러 요청 헤더나 쿼리 스트링 파라미터가 Lambda 함수로 전달될 때 값의 배열(array) 형태로 전달됩니다.
    *   예시: `name=foo`와 `name=bar`가 함께 요청되면, Lambda 이벤트에서 `name: ["foo", "bar"]`와 같이 전달됩니다.

이제 네트워킹과 Lambda 함수에 대해 이야기해 볼 차례입니다

### Lambda 함수와 VPC 네트워킹

기본적으로 Lambda 함수는 사용자의 VPC 외부, 즉 AWS가 관리하는 격리된 VPC 환경 내에서 실행됩니다. 이로 인해 Lambda 함수는 인터넷상의 공용 API나 다른 AWS 공용 서비스(예: DynamoDB, S3)에는 접근할 수 있지만, 사용자의 VPC 내부에 있는 사설 리소스(예: EC2 인스턴스, RDS 데이터베이스, ElastiCache 클러스터, 내부 로드 밸런서)에는 직접 접근할 수 없습니다.

#### 1. VPC 내 Lambda 함수 배포 (VPC Lambda)

Lambda 함수가 VPC 내의 사설 리소스와 통신해야 하는 경우, 함수를 해당 VPC에 연결하도록 구성할 수 있습니다.

*   **설정 요구 사항**:
    *   **VPC ID**: 함수를 연결할 VPC.
    *   **서브넷(Subnets)**: 함수가 사용할 VPC 내의 하나 이상의 서브넷.
    *   **보안 그룹(Security Groups)**: 함수에 적용할 보안 그룹으로, 아웃바운드 트래픽을 제어하고 VPC 내 다른 리소스의 보안 그룹에서 인바운드 규칙을 설정하는 데 사용됩니다.
*   **ENI (Elastic Network Interface) 생성**:
    *   Lambda 함수를 VPC에 연결하면, 지정된 각 서브넷에 대해 Lambda 서비스가 ENI를 생성합니다. 이 ENI는 Lambda 함수가 VPC 내에서 네트워크 주소를 가질 수 있도록 합니다.
    *   ENI 생성을 위해 Lambda 함수의 실행 역할에는 `AWSLambdaVPCAccessExecutionRole` AWS 관리형 정책 또는 동등한 권한(`ec2:CreateNetworkInterface`, `ec2:DescribeNetworkInterfaces`, `ec2:DeleteNetworkInterface`)이 필요합니다.
*   **사설 리소스 접근 (예: 사설 RDS 데이터베이스)**:
    1.  Lambda 함수는 VPC 내에 생성된 ENI를 통해 라우팅됩니다.
    2.  대상 RDS 데이터베이스의 보안 그룹은 Lambda 함수에 할당된 보안 그룹으로부터의 인바운드 트래픽(해당 데이터베이스 포트)을 허용하도록 설정되어야 합니다.

#### 2. VPC Lambda와 인터넷 및 AWS 서비스 접근

*   **인터넷 접근의 기본 제한**:
    *   **주의!**: Lambda 함수를 VPC 내에 배포하면 **기본적으로 인터넷 접근 권한이 없습니다.**
    *   **공용 서브넷에 배포해도 인터넷 접근 불가**: EC2 인스턴스와 달리, Lambda 함수는 공용 IP 주소를 할당받지 않으므로 공용 서브넷에 배포되더라도 직접 인터넷에 연결할 수 없습니다. (시험 출제 포인트)
*   **VPC Lambda에서 인터넷 접근 방법**:
    1.  Lambda 함수를 VPC 내의 **사설 서브넷**에 배포합니다.
    2.  해당 사설 서브넷의 라우팅 테이블이 **NAT 게이트웨이(NAT Gateway)** 또는 **NAT 인스턴스(NAT Instance)** (공용 서브넷에 위치)를 가리키도록 설정합니다.
    3.  NAT 디바이스는 인터넷 게이트웨이(IGW)를 통해 외부 인터넷과 통신합니다.
*   **VPC Lambda에서 AWS 공용 서비스 접근**:
    *   **NAT 게이트웨이 경유**: 위와 같이 NAT 게이트웨이가 구성되어 있다면, S3나 DynamoDB 같은 AWS 공용 서비스에도 NAT를 통해 인터넷을 거쳐 접근할 수 있습니다.
    *   **VPC 엔드포인트(VPC Endpoints) 사용**: 인터넷 게이트웨이나 NAT 디바이스를 통하지 않고 AWS 네트워크 내에서 비공개적으로 AWS 서비스에 접근하려면 VPC 엔드포인트를 사용합니다.
        *   **게이트웨이 엔드포인트**: S3 및 DynamoDB에 사용됩니다. 라우팅 테이블에 경로를 추가하여 트래픽을 엔드포인트로 보냅니다.
        *   **인터페이스 엔드포인트 (AWS PrivateLink)**: 다른 많은 AWS 서비스(API Gateway, Kinesis, SQS 등)에 사용되며, 서브넷 내에 ENI를 생성하여 프라이빗 IP 주소를 통해 서비스에 접근합니다.

#### 3. CloudWatch Logs 접근

Lambda 함수를 VPC 내 사설 서브넷에 배포하더라도, CloudWatch Logs 서비스로 로그를 전송하는 기능은 NAT 게이트웨이나 VPC 엔드포인트 설정 없이 **항상 정상적으로 작동합니다.** Lambda 서비스가 이를 백그라운드에서 처리합니다.

### Lambda 호출 방식: 동기식 호출 (Synchronous Invocation)

Lambda 함수를 호출하고 그 결과를 즉시 받아 처리하는 방식입니다. 호출한 측(클라이언트, 서비스)은 Lambda 함수의 실행이 완료될 때까지 대기합니다.

*   **호출 주체**: CLI, SDK, API Gateway, Application Load Balancer (ALB) 등
*   **특징**:
    *   호출자는 Lambda 함수의 응답(성공 또는 오류)을 직접 받습니다.
    *   오류 발생 시, 호출한 클라이언트 측에서 재시도 등의 오류 처리를 담당해야 합니다.
        *   예: Lambda 콘솔에서 테스트 실행 후 실패 시, 사용자가 직접 'Retry' 버튼을 눌러야 합니다. 클라이언트가 재시도 로직(예: 지수 백오프)을 구현할 수 있습니다.
    *   API Gateway를 통해 Lambda 함수를 호출하는 경우, 클라이언트는 API Gateway로부터 Lambda 함수의 최종 응답을 받게 됩니다.

#### Lambda를 동기식으로 호출하는 주요 AWS 서비스

*   **사용자 직접 호출 및 주요 연동 서비스**:
    *   **Application Load Balancer (ALB)**: Elastic Load Balancing을 통해 HTTP(S) 요청을 받아 Lambda 함수를 동기적으로 호출합니다.
    *   **Amazon API Gateway**: RESTful API 및 WebSocket API의 백엔드로 Lambda 함수를 동기적으로 호출합니다.
    *   **AWS Lambda@Edge**: CloudFront CDN의 엣지 로케이션에서 Lambda 함수를 동기적으로 실행하여 요청/응답을 수정합니다.
    *   **Amazon Cognito**: 사용자 풀(User Pools)의 특정 이벤트(예: 사전 가입 트리거, 사용자 지정 인증 흐름) 발생 시 Lambda 함수를 동기적으로 호출합니다.
    *   **AWS Step Functions**: 워크플로우의 특정 상태(Task 상태)에서 Lambda 함수를 동기적으로 호출하고 결과를 기다립니다.
*   **기타 서비스 (동기 호출 지원)**:
    *   **Amazon Lex**: 챗봇의 비즈니스 로직을 처리하기 위해 Lambda 함수를 동기적으로 호출합니다.
    *   **Amazon Alexa**: Alexa 스킬의 백엔드 로직을 위해 Lambda 함수를 동기적으로 호출합니다.
    *   **Amazon S3 Batch Operations**: S3 객체에 대한 대규모 배치 작업 시, 각 객체 처리를 위해 Lambda 함수를 동기적으로 호출할 수 있습니다. (작업 완료를 기다림)
    *   **Amazon Kinesis Data Firehose**: 데이터 스트림 전송 중 데이터 변환(transformation)을 위해 Lambda 함수를 동기적으로 호출합니다. 변환된 데이터를 받아야 다음 단계로 진행할 수 있습니다.
    *   기타: AWS CodeCommit (새 브랜치/태그/푸시 시), AWS CodePipeline, CloudWatch Logs (로그 처리), Amazon SES (이메일 수신 시), AWS CloudFormation, AWS Config, AWS IoT 등.

### Lambda 이벤트 소스 매핑 (Event Source Mapping)

Lambda가 이벤트를 처리하는 세 번째 주요 방식은 이벤트 소스 매핑을 사용하는 것입니다. 이는 **Kinesis Data Streams, DynamoDB Streams, Amazon SQS (Standard 및 FIFO)** 와 같은 폴링 기반 서비스에 적용됩니다.

**핵심 작동 방식:**

1.  **폴링(Polling)**: Lambda 서비스 내의 이벤트 소스 매핑이 원본 서비스(예: Kinesis 샤드, SQS 대기열)에서 레코드를 주기적으로 요청(폴링)하여 가져옵니다.
2.  **동기식 호출(Synchronous Invocation)**: 이벤트 소스 매핑은 폴링해 온 레코드들의 **배치(batch)**를 Lambda 함수로 전달하여 **동기적으로 호출**합니다. Lambda 함수는 이 배치를 처리하고 결과를 반환합니다.

이벤트 소스 매핑은 크게 스트림 기반과 대기열 기반으로 나눌 수 있습니다.

#### 1. 스트림 기반 이벤트 소스 매핑 (Kinesis Data Streams, DynamoDB Streams)

*   **레코드 처리**:
    *   이벤트 소스 매핑은 각 스트림 샤드(shard)에 대해 반복자(iterator)를 생성하고, 샤드 내의 레코드를 순차적으로 처리합니다.
    *   **읽기 시작 위치(Starting Position)** 설정 가능:
        *   `LATEST`: 가장 최근 레코드부터 읽습니다.
        *   `TRIM_HORIZON`: 샤드의 가장 오래된 레코드부터 읽습니다.
        *   `AT_TIMESTAMP`: 지정된 타임스탬프부터 읽습니다.
*   **데이터 보존**: Lambda가 레코드를 처리한 후에도 해당 레코드는 스트림에서 삭제되지 않습니다. 따라서 다른 소비자(consumer)들도 동일한 스트림 데이터를 읽을 수 있습니다.
*   **배치 처리 및 처리량 향상**:
    *   **배치 윈도우(Batch Window)**: 트래픽이 낮은 경우, 최대 5분까지 레코드를 축적했다가 한 번에 처리하여 Lambda 호출 효율성을 높일 수 있습니다.
    *   **병렬화 인수(Parallelization Factor)**: 샤드당 동시에 여러 배치(기본 1개, 최대 10개)를 병렬로 처리하도록 설정하여 처리량을 늘릴 수 있습니다. 파티션 키별 순서는 여전히 보장됩니다.
*   **오류 처리**:
    *   **기본 동작**: Lambda 함수 실행 중 오류가 발생하면, 해당 배치의 모든 레코드가 성공적으로 처리되거나 데이터 보존 기간이 만료될 때까지 재시도됩니다. 이로 인해 특정 샤드의 처리가 중단될 수 있습니다.
    *   **오류 처리 옵션**:
        *   **최대 레코드 수명(Maximum Record Age)**: 지정된 시간보다 오래된 레코드는 폐기합니다.
        *   **최대 재시도 횟수(Maximum Retry Attempts)**: 재시도 횟수를 제한합니다.
        *   **오류 시 배치 분할(Bisect batch on function error)**: 함수 시간 초과 등의 오류 발생 시, 문제가 된 배치를 둘로 나누어 더 작은 배치로 재시도합니다.
        *   **실패 시 대상(On-failure destination)**: 재시도 후에도 처리되지 않은 배치를 SQS 대기열이나 SNS 주제로 보내 추가 분석 및 처리를 할 수 있습니다 (Lambda Destinations 기능).

#### 2. 대기열 기반 이벤트 소스 매핑 (Amazon SQS)

*   **레코드 처리**:
    *   이벤트 소스 매핑은 SQS 대기열을 긴 폴링(Long Polling) 방식으로 효율적으로 폴링합니다.
    *   **배치 크기(Batch Size)**: 1개에서 최대 10개의 메시지까지 배치 크기를 구성할 수 있습니다 (표준 SQS 대기열의 경우 최대 10,000개까지 가능하나, Lambda의 이벤트 소스 매핑에서는 다름).
*   **오류 처리 및 메시지 관리**:
    *   **메시지 반환**: Lambda 함수가 메시지 처리에 실패하면, 해당 메시지(또는 배치)는 SQS 대기열의 가시성 제한 시간(Visibility Timeout) 이후 다시 대기열로 돌아가 다른 Lambda 함수 인스턴스에 의해 처리될 수 있습니다.
    *   **가시성 제한 시간 권장 사항**: Lambda 함수의 시간 제한보다 최소 6배 길게 설정하는 것이 좋습니다.
    *   **DLQ (Dead-Letter Queue) 설정**: Lambda 함수가 아닌 **SQS 대기열 자체에 DLQ를 설정**해야 합니다. Lambda DLQ는 비동기 호출 전용이며, 이벤트 소스 매핑은 Lambda 함수를 동기적으로 호출하기 때문입니다. 또는 Lambda Destinations 기능을 사용할 수 있습니다.
    *   **메시지 삭제**: Lambda 함수가 성공적으로 배치를 처리하면, 이벤트 소스 매핑이 해당 메시지들을 SQS 대기열에서 삭제합니다.
    *   **멱등성 중요**: 드물지만, 함수 오류가 발생하지 않았음에도 불구하고 이벤트 소스 매핑이 SQS 대기열에서 동일한 메시지를 두 번 이상 수신할 수 있습니다. 따라서 Lambda 함수는 멱등성(멱등하게)을 가지도록 설계해야 합니다.
*   **SQS FIFO (First-In, First-Out) 대기열**:
    *   **순차 처리**: 메시지 그룹 ID(Message Group ID)를 기준으로 메시지 순서가 보장됩니다.
    *   **스케일링**: 활성 메시지 그룹의 수만큼 Lambda 함수가 확장됩니다.
*   **SQS 표준 대기열**:
    *   **순서 미보장**: 메시지 처리 순서가 보장되지 않습니다.
    *   **높은 확장성**: Lambda는 대기열의 메시지를 가능한 한 빨리 처리하기 위해 신속하게 확장됩니다(예: 분당 60개 이상의 인스턴스 추가, 동시에 최대 1,000개 배치 처리 가능).

#### 이벤트 소스 매핑 스케일링 요약

*   **Kinesis Data Streams & DynamoDB Streams**:
    *   기본: 스트림의 각 샤드당 하나의 Lambda 호출이 동시에 실행됩니다.
    *   병렬화 인수 사용 시: 샤드당 최대 10개의 배치를 동시에 처리할 수 있습니다.
*   **SQS 표준 대기열**:
    *   매우 빠르게 확장됩니다 (분당 +60 인스턴스, 최대 동시 1,000 배치/초).
*   **SQS FIFO 대기열**:
    *   활성 메시지 그룹의 수에 따라 Lambda 함수 인스턴스 수가 결정됩니다.

이벤트 소스 매핑은 특히 스트리밍 데이터 처리나 대규모 메시지 처리 작업에 유용하며, 각 서비스의 특성에 맞는 구성과 오류 처리 전략을 수립하는 것이 중요합니다.

### Lambda 이벤트 소스 매핑 실습

이벤트 소스 매핑의 실제 작동 방식을 SQS를 중심으로 살펴보고, Kinesis 데이터 스트림 연동 시의 주요 설정 옵션도 확인합니다.

#### 1. SQS 이벤트 소스 매핑 실습

**가. 사전 준비**

1.  **Lambda 함수 생성**:
    *   이름: `lambda-sqs` (예시)
    *   런타임: Python 3.8 (예시) 또는 원하는 런타임
2.  **SQS 대기열 생성**:
    *   이름: `lambda-demo-sqs` (예시)
    *   유형: 표준(Standard) 대기열

**나. Lambda 트리거 추가 및 권한 설정**

1.  생성한 `lambda-sqs` 함수에서 **[트리거 추가(Add trigger)]**를 선택합니다.
2.  트리거 구성에서 소스로 **SQS**를 선택합니다.
3.  SQS 대기열로 위에서 생성한 `lambda-demo-sqs`를 지정합니다.
4.  **배치 크기(Batch size)**: 한 번의 Lambda 함수 호출로 처리할 최대 메시지 수 (예: 기본값 10).
5.  **배치 윈도우(Batch window)**: 함수를 호출하기 전 최대 레코드 수집 대기 시간(초).
6.  **[활성화(Enable trigger)]**가 체크되어 있는지 확인하고 **[추가(Add)]**를 클릭합니다.
    *   **초기 권한 오류**: 이 단계에서 Lambda 함수의 실행 역할(Execution Role)에 SQS 대기열에서 메시지를 읽을 권한(`sqs:ReceiveMessage`, `sqs:DeleteMessage`, `sqs:GetQueueAttributes`)이 없다는 오류가 발생할 수 있습니다.
7.  **권한 문제 해결**:
    *   Lambda 함수의 **[구성(Configuration)] > [권한(Permissions)]** 탭으로 이동하여 역할 이름을 클릭, IAM 콘솔로 이동합니다.
    *   해당 역할에 **`AWSLambdaSQSQueueExecutionRole`** AWS 관리형 정책을 연결(Attach)합니다. 이 정책에는 SQS 대기열 폴링에 필요한 권한들이 포함되어 있습니다.
    *   다시 Lambda 함수로 돌아와 SQS 트리거를 추가하면 성공적으로 생성됩니다.

**다. Lambda 함수 코드 수정**

SQS로부터 전달받는 이벤트 내용을 확인하기 위해 Lambda 함수 코드를 다음과 같이 간단히 수정합니다.

```python
import json

def lambda_handler(event, context):
    print("Received event: " + json.dumps(event, indent=2))
    # 메시지 처리 로직 (예시로 단순 출력)
    for record in event.get('Records', []):
        print("Message Body: " + record.get('body'))
        # 실제 애플리케이션에서는 여기서 메시지 내용을 기반으로 작업 수행
    return {
        'statusCode': 200,
        'body': json.dumps('Successfully processed messages')
    }
```

수정 후 **[배포(Deploy)]**합니다.

**라. 테스트 및 확인**

1.  **SQS 대기열에 메시지 전송**:
    *   SQS 콘솔에서 `lambda-demo-sqs` 대기열을 선택하고 **[메시지 보내기 및 받기(Send and receive messages)]**를 클릭합니다.
    *   메시지 본문(Message body)에 `Hello World!`와 같이 테스트 내용을 입력합니다.
    *   (선택 사항) 메시지 속성(Message attributes)도 추가해 볼 수 있습니다 (예: 이름 `foo`, 유형 `String`, 값 `bar`).
    *   메시지를 전송합니다.
2.  **CloudWatch Logs 확인**:
    *   Lambda 함수의 **[모니터(Monitor)] > [CloudWatch 로그 보기(View CloudWatch logs)]**로 이동합니다.
    *   최신 로그 스트림을 열어보면 Lambda 함수가 SQS로부터 받은 이벤트의 전체 구조(레코드 목록, 각 레코드의 body, messageAttributes, eventSource 등)를 확인할 수 있습니다.
3.  **SQS 대기열 확인**:
    *   다시 SQS 콘솔의 **[메시지 보내기 및 받기(Send and receive messages)]**에서 **[메시지 폴링(Poll for messages)]**을 해보면, 사용 가능한 메시지가 0개로 나타나야 합니다. 이는 Lambda 함수가 메시지를 성공적으로 처리하고 대기열에서 삭제했음을 의미합니다.

**마. 리소스 정리**

실습이 끝나면 불필요한 Lambda 함수 호출과 SQS 폴링으로 인한 비용 발생을 막기 위해 Lambda 함수에 설정된 SQS 트리거를 **비활성화(Disable)**하거나 **삭제(Delete)**합니다.

#### 2. Kinesis 이벤트 소스 매핑 (주요 설정 옵션 소개)

Lambda 함수에 Kinesis Data Streams를 이벤트 소스로 추가할 때 설정할 수 있는 주요 옵션들은 다음과 같습니다 (실제 생성은 하지 않고 옵션만 확인).

1.  **Kinesis 스트림**: 연결할 Kinesis 데이터 스트림을 선택합니다.
2.  **소비자(Consumer)**: 표준 소비자 또는 향상된 팬아웃(Enhanced fan-out) 소비자 중에서 선택할 수 있습니다. 향상된 팬아웃은 각 샤드에 대해 전용 처리량을 제공하여 여러 애플리케이션이 동시에 스트림을 읽을 때 유용합니다.
3.  **배치 크기(Batch size)**: 한 번의 호출로 Lambda 함수에 전달할 최대 레코드 수 (예: 기본값 100).
4.  **배치 윈도우(Batch window)**: 함수를 호출하기 전 최대 레코드 수집 대기 시간(초).
5.  **시작 위치(Starting position)**:
    *   `Latest`: 스트림의 가장 최신 레코드부터 읽기 시작합니다.
    *   `Trim horizon`: 스트림의 가장 오래된(만료되지 않은) 레코드부터 읽기 시작합니다.
    *   `At timestamp`: 지정된 타임스탬프 이후의 레코드부터 읽기 시작합니다.
6.  **추가 설정(Additional settings)**:
    *   **실패 시 대상(On-failure destination)**: 처리 실패한 레코드를 SQS 또는 SNS로 보냅니다 (Lambda Destinations).
    *   **재시도 횟수(Retry attempts)**: 최대 재시도 횟수를 지정합니다.
    *   **최대 레코드 수명(Maximum record age)**: 이 시간보다 오래된 레코드는 폐기합니다.
    *   **오류 시 배치 분할(Bisect batch on function error)**: 함수 시간 초과 등의 오류 시 배치를 더 작게 나누어 재시도합니다.
    *   **샤드당 동시 배치(Concurrent batches per shard)**: 샤드당 병렬로 처리할 배치 수 (기본 1, 최대 10). 파티션 키별 순서는 보장됩니다.
    *   **텀블링 윈도우 기간(Tumbling window duration)**: 상태 저장 집계를 위해 윈도우를 설정합니다 (초 단위).
    *   **배치 항목 실패 보고(Report batch item failures)**: 배치 내 특정 레코드만 실패 처리할 수 있게 합니다. (SQS, Kinesis, DynamoDB Streams 지원)

**참고**: 시험 대비로는 Kinesis가 Lambda 함수의 이벤트 소스 매퍼로 사용될 수 있다는 점과 주요 설정(시작 위치, 배치 크기, 오류 처리 옵션 등)의 개념을 이해하는 것이 중요합니다. 모든 세부 옵션을 암기할 필요는 없습니다.

### Lambda 함수 핸들러: 이벤트 및 컨텍스트 객체

Lambda 함수가 호출될 때, Lambda 런타임은 핸들러 함수에 두 가지 주요 객체를 전달합니다: `이벤트(event)` 객체와 `컨텍스트(context)` 객체입니다.

*   **이벤트 객체 (Event Object)**:
    *   Lambda 함수를 트리거한 서비스로부터 전달되는 데이터(페이로드)입니다. 일반적으로 JSON 문서 형태입니다.
    *   이벤트 객체의 내용은 어떤 서비스가 함수를 호출했는지, 어떤 이벤트가 발생했는지에 따라 달라집니다. 예를 들어, S3 이벤트의 경우 버킷 이름과 객체 키 정보가 포함되고, API Gateway 호출의 경우 HTTP 요청 파라미터 등이 포함됩니다.
    *   사용하는 런타임에 따라 특정 타입으로 변환될 수 있습니다 (예: Python에서는 딕셔너리, Node.js에서는 JavaScript 객체).
*   **컨텍스트 객체 (Context Object)**:
    *   호출, 함수 자체, 그리고 실행 환경에 대한 런타임 정보를 제공하는 객체입니다.
    *   컨텍스트 객체를 통해 다음과 같은 정보를 얻을 수 있습니다:
        *   AWS 요청 ID (`aws_request_id`)
        *   호출된 함수의 ARN (`invoked_function_arn`)
        *   함수 이름 (`function_name`), 버전 (`function_version`)
        *   CloudWatch 로그 그룹 이름 (`log_group_name`), 로그 스트림 이름 (`log_stream_name`)
        *   함수에 할당된 메모리 제한 (`memory_limit_in_mb`)
        *   함수 실행에 남은 시간 (`get_remaining_time_in_millis()` 메소드)

**예시 (Python 핸들러):**
```python
import json
import os # 컨텍스트 객체 예시를 위해 추가 (실제 사용은 아래 환경변수 섹션 참조)

def lambda_handler(event, context):
    print("## EVENT")
    print(json.dumps(event, indent=2))
    
    print("## CONTEXT")
    print(f"AWS Request ID: {context.aws_request_id}")
    print(f"Function Name: {context.function_name}")
    # 기타 컨텍스트 정보 출력 가능
    
    return {
        'statusCode': 200,
        'body': json.dumps('Event and context processed')
    }
```
이벤트 객체는 주로 함수가 처리해야 할 입력 데이터를 담고 있으며, 컨텍스트 객체는 함수 실행 자체를 관리하거나 로깅, 모니터링에 필요한 메타데이터를 제공합니다.

### Lambda Destinations

Lambda Destinations는 비동기(Asynchronous) 호출 및 이벤트 소스 매핑(스트림 기반: Kinesis, DynamoDB Streams)의 실행 결과를 다른 AWS 서비스로 라우팅하는 기능입니다.

*   **기존 DLQ (Dead-Letter Queue)와의 관계**:
    *   비동기 호출의 경우, Destinations는 DLQ보다 더 유연하며 성공/실패 모두에 대해 다양한 대상으로 라우팅 가능하여 권장됩니다.
*   **비동기 호출의 Destinations**:
    *   **조건**: `성공 시(On success)` 또는 `실패 시(On failure)`.
    *   **대상**: SQS, SNS, 다른 Lambda 함수, EventBridge 이벤트 버스.
*   **이벤트 소스 매핑(스트림)의 Destinations**:
    *   **조건**: 처리 실패 후 폐기된(discarded) 이벤트 배치에 대해 `실패 시(On-failure)`.
    *   **대상**: SQS, SNS.

### Lambda Destinations 실습 (비동기 호출 - S3 트리거)

S3 이벤트로 비동기 호출되는 Lambda 함수의 성공 및 실패 결과를 각각 다른 SQS 대기열로 보내는 실습입니다.

**1. 사전 준비**

*   S3 이벤트 알림을 받을 Lambda 함수 (예: `lambda-s3-dest-test`).
*   성공 알림용 SQS 표준 대기열 (예: `s3-lambda-success-dest`).
*   실패 알림용 SQS 표준 대기열 (예: `s3-lambda-failure-dest`).

**2. Lambda Destinations 설정**

*   Lambda 함수 구성의 [대상(Destinations)] 탭 또는 [비동기식 호출(Asynchronous invocation)] 섹션에서 [대상 추가(Add destination)]를 클릭합니다.
*   **실패 시(On failure) 조건**: 소스 `비동기식 호출`, 대상 `SQS queue` (`s3-lambda-failure-dest`).
*   **성공 시(On success) 조건**: 소스 `비동기식 호출`, 대상 `SQS queue` (`s3-lambda-success-dest`).
*   Lambda 콘솔이 저장 시 필요한 IAM 권한을 실행 역할에 자동 추가합니다.

**3. 성공 시나리오 테스트**

*   Lambda 함수가 정상 성공하도록 코드를 유지하고 S3에 파일을 업로드합니다.
*   `s3-lambda-success-dest` SQS 대기열에서 성공 메시지(요청 컨텍스트, 이벤트, 응답 페이로드 포함)를 확인합니다.

**4. 실패 시나리오 테스트**

*   Lambda 함수 코드를 수정하여 의도적으로 예외를 발생시킵니다 (예: `raise Exception("Intentional failure!")`).
*   S3에 다른 파일을 업로드합니다.
*   비동기 호출의 자동 재시도 후, `s3-lambda-failure-dest` SQS 대기열에서 실패 메시지(실패 원인, 이벤트, 오류 응답 포함)를 확인합니다.

### Lambda 실행 역할 및 권한

Lambda 함수는 AWS 서비스와 상호작용하기 위해 IAM 역할을 사용합니다.

*   **실행 역할 (Execution Role)**:
    *   모든 Lambda 함수에 연결되어 함수가 AWS 서비스에 액세스하는 데 필요한 권한을 정의합니다.
    *   **AWS 관리형 정책 예시**: 
        * `AWSLambdaBasicExecutionRole` (CloudWatch Logs): Lambda 함수가 CloudWatch Logs에 로그를 쓸 수 있는 기본 권한을 제공합니다. AWS Management Console을 사용하여 Lambda 함수를 생성할 때 자동으로 생성되는 실행 역할에 이 정책이 포함되는 경우가 많습니다. 하지만 CloudFormation 템플릿이나 기타 Infrastructure as Code(IAC) 메서드를 통해 Lambda 함수를 정의할 때는 함수의 실행 역할에 이 정책을 명시적으로 연결하여 적절한 로깅 권한을 부여해야 할 수 있습니다.
        * `AWSLambdaVPCAccessExecutionRole` (VPC): VPC 내에서 Lambda 함수를 실행할 때 필요한 권한을 제공합니다.
        * `AWSLambdaSQSQueueExecutionRole` (SQS 폴링): SQS 대기열에서 메시지를 읽고 처리하는 데 필요한 권한을 제공합니다.
        * `AWSXRayDaemonWriteAccess` (X-Ray): AWS X-Ray를 사용한 추적에 필요한 권한을 제공합니다.
    *   이벤트 소스 매핑 시 해당 소스에서 데이터를 읽고 관리할 권한이 필요합니다.
*   **리소스 기반 정책 (Resource-based Policy)**:
    *   다른 AWS 서비스나 계정이 해당 Lambda 함수를 호출할 수 있도록 허용하는 권한입니다.
    *   S3 등의 서비스가 Lambda를 호출할 때 필요하며, 콘솔에서 트리거 설정 시 자동 생성되는 경우가 많습니다.
*   **권장 사항**: 각 함수에는 필요한 최소한의 권한만 가진 전용 실행 역할을 생성합니다.

### Lambda 환경 변수

코드 수정 없이 함수 설정을 외부에서 관리하고 동작을 조정하는 기능입니다.

*   **정의**: 함수 구성의 일부로 저장되는 문자열 형식의 키-값 쌍.
*   **활용**: 데이터베이스 연결 정보, API 엔드포인트, 기능 플래그 등.
*   **보안**: KMS를 사용하여 민감한 환경 변수를 암호화할 수 있습니다 (전송 중/저장 시 암호화).

**환경 변수 실습**

1.  **Lambda 함수 생성**: 이름 `lambda-config-demo` (예시), Python 런타임.
2.  **코드 수정**: `os.getenv("ENVIRONMENT_NAME")`를 사용하여 환경 변수 값을 읽어 반환하도록 합니다.
    ```python
    import os
    import json

    def lambda_handler(event, context):
        env_name = os.getenv("ENVIRONMENT_NAME")
        print(f"ENVIRONMENT_NAME is: {env_name}")
        return {
            'statusCode': 200,
            'body': json.dumps(env_name) # 환경 변수 값을 직접 반환
        }
    ```
    코드를 배포합니다.
3.  **환경 변수 설정**:
    *   Lambda 함수 **[구성(Configuration)] > [환경 변수(Environment variables)]**에서 [편집(Edit)] 클릭.
    *   키 `ENVIRONMENT_NAME`, 값 `dev`를 추가하고 저장합니다. (암호화는 본 실습에서 다루지 않음)
4.  **테스트**: Lambda 콘솔의 [Test] 탭에서 테스트 이벤트를 생성하고 실행합니다. 응답으로 `"dev"`가 반환되는 것을 확인합니다.
5.  **값 변경 및 재테스트**: 환경 변수 값을 `prod`로 변경하고 다시 테스트하면, 코드 변경 없이 응답이 `"prod"`로 바뀌는 것을 확인할 수 있습니다.

---

[실습]

1.  **람다 초기화 관련 사항**:
    *   람다 함수가 처음 호출될 때 또는 한동안 호출되지 않았다가 다시 호출될 때 초기화 과정(init code)이 발생합니다.
    *   이 초기화 시간은 핸들러 함수 외부에서 수행되며, SDK 클라이언트 초기화, 데이터베이스 커넥션 설정 등을 이 단계에서 수행하여 실제 요청 처리 시간을 줄일 수 있습니다.
2.  **CloudShell을 이용한 실습 단계**:
    *   `aws lambda list-functions`: 현재 리전의 람다 함수 목록을 확인합니다.
    *   람다 함수를 인터넷에 노출시키는 방법:
        *   Application Load Balancer (ALB) 사용
        *   API Gateway 사용

### ALB와 Lambda 연동 시 고려사항

*   **HTTP에서 JSON으로 변환**: ALB가 Lambda로 요청을 전달할 때 HTTP 요청은 특정 구조의 JSON 문서로 변환됩니다.
*   **ALB Multi Header Values 지원**: (시험 출제 포인트!)
    *   ALB 설정에서 다중 헤더 값(Multi-Header Values)을 활성화할 수 있습니다.
    *   활성화 시, 동일한 키를 가진 여러 요청 헤더나 쿼리 스트링 파라미터가 Lambda 함수로 전달될 때 값의 배열(array) 형태로 전달됩니다.
    *   예시: `name=foo`와 `name=bar`가 함께 요청되면, Lambda 이벤트에서 `name: ["foo", "bar"]`와 같이 전달됩니다.

이제 네트워킹과 Lambda 함수에 대해 이야기해 볼 차례입니다

### Lambda 함수와 VPC 네트워킹

기본적으로 Lambda 함수는 사용자의 VPC 외부, 즉 AWS가 관리하는 격리된 VPC 환경 내에서 실행됩니다. 이로 인해 Lambda 함수는 인터넷상의 공용 API나 다른 AWS 공용 서비스(예: DynamoDB, S3)에는 접근할 수 있지만, 사용자의 VPC 내부에 있는 사설 리소스(예: EC2 인스턴스, RDS 데이터베이스, ElastiCache 클러스터, 내부 로드 밸런서)에는 직접 접근할 수 없습니다.

#### 1. VPC 내 Lambda 함수 배포 (VPC Lambda)

Lambda 함수가 VPC 내의 사설 리소스와 통신해야 하는 경우, 함수를 해당 VPC에 연결하도록 구성할 수 있습니다.

*   **설정 요구 사항**:
    *   **VPC ID**: 함수를 연결할 VPC.
    *   **서브넷(Subnets)**: 함수가 사용할 VPC 내의 하나 이상의 서브넷.
    *   **보안 그룹(Security Groups)**: 함수에 적용할 보안 그룹으로, 아웃바운드 트래픽을 제어하고 VPC 내 다른 리소스의 보안 그룹에서 인바운드 규칙을 설정하는 데 사용됩니다.
*   **ENI (Elastic Network Interface) 생성**:
    *   Lambda 함수를 VPC에 연결하면, 지정된 각 서브넷에 대해 Lambda 서비스가 ENI를 생성합니다. 이 ENI는 Lambda 함수가 VPC 내에서 네트워크 주소를 가질 수 있도록 합니다.
    *   ENI 생성을 위해 Lambda 함수의 실행 역할에는 `AWSLambdaVPCAccessExecutionRole` AWS 관리형 정책 또는 동등한 권한(`ec2:CreateNetworkInterface`, `ec2:DescribeNetworkInterfaces`, `ec2:DeleteNetworkInterface`)이 필요합니다.
*   **사설 리소스 접근 (예: 사설 RDS 데이터베이스)**:
    1.  Lambda 함수는 VPC 내에 생성된 ENI를 통해 라우팅됩니다.
    2.  대상 RDS 데이터베이스의 보안 그룹은 Lambda 함수에 할당된 보안 그룹으로부터의 인바운드 트래픽(해당 데이터베이스 포트)을 허용하도록 설정되어야 합니다.

#### 2. VPC Lambda와 인터넷 및 AWS 서비스 접근

*   **인터넷 접근의 기본 제한**:
    *   **주의!**: Lambda 함수를 VPC 내에 배포하면 **기본적으로 인터넷 접근 권한이 없습니다.** 공용 서브넷에 배포하더라도 공용 IP를 할당받지 않아 인터넷 연결이 불가능합니다. (시험 출제 포인트)
    *   **인터넷 접근 방법**: Lambda 함수를 사설 서브넷에 배포하고, 해당 서브넷의 라우팅 테이블이 공용 서브넷에 위치한 NAT 게이트웨이나 NAT 인스턴스를 가리키도록 설정해야 합니다.
    *   **AWS 공용 서비스 접근**:
        *   NAT 게이트웨이를 통해 인터넷을 거쳐 접근할 수 있습니다.
        *   VPC 엔드포인트(게이트웨이 엔드포인트: S3, DynamoDB / 인터페이스 엔드포인트: 다수 서비스)를 사용하면 NAT 없이 AWS 네트워크 내에서 비공개적으로 서비스에 접근할 수 있습니다.

3.  **CloudWatch Logs 접근**:
    *   VPC 내 배포 여부와 관계없이, Lambda 함수 로그는 항상 CloudWatch Logs로 정상 전송됩니다 (NAT 게이트웨이나 VPC 엔드포인트 불필요).

### VPC에서의 Lambda 실습 요약

*   `lambda-vpc` 함수를 생성하고, EC2 콘솔에서 `lambda-sg` 보안 그룹을 만듭니다.
*   Lambda 함수 설정에서 VPC, 3개의 서브넷, 그리고 `lambda-sg` 보안 그룹을 연결합니다.
*   초기에는 ENI 생성 권한 부족으로 실패할 수 있습니다. Lambda 함수의 실행 역할에 `LambdaENIManagementAccess` 정책(또는 `AWSLambdaVPCAccessExecutionRole`)을 연결하여 권한을 부여합니다.
*   설정 저장 후, 테스트를 실행하면 성공합니다. EC2 콘솔의 네트워크 인터페이스 메뉴에서 Lambda 함수용으로 생성된 3개의 ENI를 확인할 수 있으며, 이는 각기 다른 AZ의 서브넷에 위치하여 VPC 통신을 가능하게 합니다.

### Lambda 함수 구성과 성능

1.  **RAM**:
    *   128MB부터 10GB까지 1MB 단위로 조절 가능합니다.
    *   RAM 할당량을 늘리면 할당되는 vCPU 성능도 향상됩니다 (1,792MB RAM 시 1 vCPU).
    *   CPU 집약적인 작업의 실행 시간을 줄이려면 RAM을 늘리는 것이 효과적입니다.

2.  **타임아웃 (Timeout)**:
    *   기본값은 3초이며, 최대 900초(15분)까지 설정 가능합니다.
    *   15분을 초과하는 작업은 Lambda에 적합하지 않으며, Fargate, ECS, EC2 등을 고려해야 합니다.

3.  **실행 컨텍스트 (Execution Context)**:
    *   Lambda 코드가 실행되는 임시 런타임 환경입니다.
    *   데이터베이스 연결, SDK 클라이언트 초기화 등을 핸들러 함수 외부(실행 컨텍스트)에서 수행하면, 연속된 호출 시 해당 리소스를 재사용하여 성능을 향상시킬 수 있습니다 (콜드 스타트 이후 웜 스타트).
    *   `/tmp` 디렉토리: 함수 실행 중 임시 파일을 저장할 수 있는 공간을 제공하며, 최대 10GB까지 사용 가능합니다. 이 공간의 내용은 실행 컨텍스트가 유지되는 동안 지속됩니다.

4.  **성능 최적화 예시**:
    *   **비효율적인 코드**: Lambda 핸들러 함수 내에서 매번 DB 연결(`db.connect()`)을 수행하면 호출마다 연결 시간이 소요됩니다.
    *   **모범 사례**: DB 연결 객체(`db_client`)를 핸들러 함수 바깥(글로벌 스코프)에서 한 번만 초기화하고, 핸들러 함수 내에서는 이 객체를 재사용합니다. 이렇게 하면 초기화에 오래 걸리는 작업을 여러 호출에 걸쳐 공유하여 성능을 크게 개선할 수 있습니다.

### Lambda 함수 구성 및 성능 실습 요약

*   `lambda-config-demo` 함수를 사용합니다.
*   **메모리 설정**: 128MB ~ 10,240MB 범위에서 설정 가능하며, 메모리가 늘면 CPU도 비례하여 증가합니다. 비용 최적화를 위해 실제 필요한 만큼만 할당하는 것이 중요합니다.
*   **타임아웃 테스트**:
    *   기본 타임아웃 3초에서 `time.sleep(2)`는 성공하지만, `time.sleep(5)`는 "Task timed out after 3.00 seconds" 오류 발생.
    *   타임아웃을 6초로 늘리면 `time.sleep(5)`도 성공합니다.
    *   타임아웃은 너무 길게 설정하면 함수가 멈췄을 때 오류 분석까지 오래 기다려야 하므로, 합리적으로 설정해야 합니다.
*   **실행 컨텍스트 최적화 테스트**:
    *   `connect_to_db()` 함수(내부적으로 `time.sleep(3)`으로 3초 소요 가정)를 핸들러 내부에 위치시키면, 매 테스트 호출마다 약 3초의 실행 시간(Duration)이 소요됩니다.
    *   `connect_to_db()` 함수 정의와 호출(결과를 글로벌 변수에 저장)을 핸들러 외부에 위치시키면, 첫 호출 시에는 초기화 시간(Init Duration)으로 약 3초가 소요되지만, 이후 호출부터는 Duration이 1ms 미만으로 매우 짧아집니다. 이는 실행 컨텍스트가 재사용되어 DB 연결이 다시 수행되지 않기 때문입니다.

### Lambda 계층 (Layers)

Lambda 계층은 두 가지 주요 기능을 제공합니다:

1.  **사용자 지정 런타임 생성**: 기본 지원 언어가 아니더라도 (예: C++, Rust) 커뮤니티나 사용자가 직접 런타임을 만들어 Lambda에서 사용할 수 있게 합니다.
2.  **종속성 외부화 및 재사용**:
    *   Lambda 함수 코드와 공통 라이브러리/종속성을 분리합니다.
    *   애플리케이션 패키지(배포 파일)의 크기를 줄여 배포 속도를 향상시킵니다.
    *   자주 변경되지 않는 무거운 라이브러리를 계층으로 만들어 여러 Lambda 함수에서 공유하고 재사용할 수 있습니다.

### Lambda 계층 실습 요약

*   `lambda-layer-demo` 함수 (Python 3.8)를 생성합니다.
*   계층 추가 옵션에서 "AWS 계층"을 선택하고, `AWSLambda-Python38-SciPy1x` (SciPy 라이브러리가 포함된 AWS 제공 계층)를 최신 버전으로 추가합니다.
*   제공된 샘플 Python 코드를 Lambda 함수에 붙여넣습니다. 이 코드는 `numpy`와 `scipy.spatial`을 임포트합니다.
*   함수를 배포하고 테스트하면, 코드 패키지에 직접 라이브러리를 포함하지 않았음에도 불구하고 계층을 통해 `SciPy` 라이브러리가 성공적으로 로드되어 코드가 실행되는 것을 확인할 수 있습니다.

### 파일 시스템 마운팅 (Lambda와 Amazon EFS)

*   VPC 내에서 실행되는 Lambda 함수는 Amazon EFS(Elastic File System) 파일 시스템에 액세스할 수 있습니다.
*   EFS 파일 시스템은 Lambda 초기화 과정에서 로컬 디렉터리에 마운트되도록 구성됩니다.
*   이 기능을 사용하려면 EFS 액세스 포인트가 필요하며, Lambda 함수는 EFS와 동일한 VPC 내의 프라이빗 서브넷에 배포되어야 합니다.
*   **한계점**: 각 Lambda 인스턴스가 EFS에 연결되므로 연결 제한에 도달하지 않도록 주의해야 하며, 동시 실행 Lambda 함수가 급증하면 연결 버스트 제한에 도달할 수도 있습니다.

### Lambda의 스토리지 옵션 비교

| 특징          | 임시 스토리지 (`/tmp`)                                 | Lambda 계층 (Layers)                                    | Amazon S3                                           | Amazon EFS                                                 |
| :------------ | :--------------------------------------------------- | :------------------------------------------------------ | :-------------------------------------------------- | :--------------------------------------------------------- |
| **크기**      | 최대 10GB                                             | 최대 250MB (압축 해제 후, 함수당 5개 제한)                 | 제한 없음 (사실상)                                   | 탄력적                                                     |
| **지속성**    | 임시 (실행 컨텍스트 소멸 시 사라짐)                        | 영구적 (계층 버전 업데이트 전까지)                         | 영구적                                                | 영구적                                                     |
| **가변성**    | 동적 (읽기/쓰기 가능)                                  | 불변 (읽기 전용)                                         | 동적 (객체 단위 읽기/쓰기/삭제)                        | 동적 (파일 시스템 작업)                                     |
| **유형**      | 파일 시스템                                            | 아카이브 (배포 시점에 로드)                                | 객체 스토리지                                         | 파일 시스템 (NFS)                                          |
| **가격**      | 512MB까지 무료, 초과 시 추가 비용 (함수 가격에 포함) | Lambda 함수 가격에 포함                                  | S3 스토리지, 요청, 데이터 전송 요금                     | EFS 스토리지, 데이터 전송, 처리량 요금                        |
| **접근 권한** | 함수 내에서만 접근                                     | IAM 권한 필요 (다른 계정/서비스에서 사용 시)               | IAM 권한 필요                                         | Lambda 실행 역할에 VPC/EFS 접근 권한 필요                 |
| **속도**      | 매우 빠름 (로컬 디스크)                                 | 매우 빠름 (로컬에 로드됨)                                 | 빠름 (네트워크 기반)                                   | 매우 빠름 (네트워크 파일 시스템, 로컬 마운트)               |
| **공유**      | Lambda 함수 호출 간 비공유                             | Lambda 함수 호출 간 공유 (동일 계층 참조 시)               | Lambda 함수 호출 간 공유 (외부 스토리지)               | Lambda 함수 호출 간 공유 (마운트된 파일 시스템)               |

### Lambda 동시성 및 스로틀링

*   **Lambda 확장성과 동시성**: Lambda 함수는 호출량에 따라 매우 빠르고 쉽게 확장되어 동시에 많은 수의 함수가 실행될 수 있습니다 (최대 1,000개까지 자동 확장 가능).
*   **예약된 동시성 (Reserved Concurrency)**:
    *   함수 수준에서 최대 동시 실행 수를 제한하는 기능입니다. 예를 들어, 특정 함수에 최대 50개의 동시 실행만 허용하도록 설정할 수 있습니다.
    *   설정된 제한을 초과하는 호출은 **스로틀(Throttle)**됩니다.
        *   **동기식 호출 시**: `ThrottleError-429` 오류가 반환됩니다.
        *   **비동기식 호출 시**: Lambda가 자동으로 재시도한 후, 실패 시 DLQ(Dead-Letter Queue)로 메시지를 보냅니다.
    *   **중요**: 예약된 동시성을 설정하지 않으면, 계정 내 모든 함수가 동시성 한도를 공유합니다. 이로 인해 하나의 함수가 과도한 동시성을 사용하면 다른 중요한 함수들까지 스로틀링될 수 있습니다. (시험 출제 포인트)
*   **동시성 한도 상향**: 기본 한도(일반적으로 계정당 1,000개) 이상의 동시 실행이 필요한 경우, AWS 지원팀에 요청하여 한도를 높일 수 있습니다.

### 비동기 호출에서의 동시성 관리

*   S3 이벤트 알림과 같이 비동기적으로 Lambda 함수가 호출될 때, 동시에 많은 이벤트가 발생하면 동시성 한도에 도달하여 스로틀링이 발생할 수 있습니다.
*   비동기 호출이 스로틀되거나 시스템 오류(500 시리즈) 발생 시, Lambda는 해당 이벤트를 내부 이벤트 대기열로 반환합니다.
*   Lambda는 최대 6시간 동안 해당 함수를 실행하려고 **자동으로 재시도**합니다. 재시도 간격은 **지수 백오프(Exponential Backoff)** 방식을 따르며, 최소 1초에서 최대 5분까지 증가합니다.

### 콜드 스타트 (Cold Start) 와 프로비저닝된 동시성 (Provisioned Concurrency)

*   **콜드 스타트**:
    *   새로운 Lambda 함수 인스턴스가 처음 생성될 때 발생하는 현상입니다.
    *   이 과정에는 코드 로딩, 런타임 환경 초기화, 핸들러 함수 외부의 초기화 코드(init 코드) 실행 등이 포함됩니다.
    *   초기화 코드가 무겁거나 종속성이 많으면 이 과정에 시간이 오래 걸려, 새 인스턴스로 처리되는 첫 번째 요청은 이미 실행 중인 인스턴스(웜 스타트)보다 지연 시간이 길어질 수 있습니다. 이는 사용자 경험에 부정적인 영향을 줄 수 있습니다.
*   **프로비저닝된 동시성**:
    *   함수가 실제로 호출되기 전에 미리 일정 수의 Lambda 실행 환경을 초기화하여 준비 상태로 유지하는 기능입니다.
    *   이를 통해 해당 동시성만큼은 콜드 스타트 없이 즉시 요청을 처리할 수 있어, 모든 호출의 지연 시간을 일관되게 낮출 수 있습니다.
    *   Application Auto Scaling을 사용하여 예약된 일정이나 사용량에 따라 프로비저닝된 동시성의 수를 자동으로 조절할 수 있습니다.
*   **VPC 콜드 스타트 개선**: 2019년 10~11월 이후로 VPC 내에서 Lambda 함수를 실행할 때 발생하던 콜드 스타트 시간이 크게 개선되었습니다.

제공된 내용에는 예약된 동시성과 프로비저닝된 동시성을 비교하는 표에 대한 참조도 언급되어 있습니다.
<!-- https://docs.aws.amazon.com/ko_kr/lambda/latest/dg/configuration-concurrency.html -->

## Lambda 동시성 설정, 종속성 관리, CloudFormation 배포, 컨테이너 이미지 지원

### Lambda 함수 동시성 설정 실습 요약

*   Lambda 함수 **[구성(Configuration)] > [동시성(Concurrency)]** 탭에서 동시성 설정을 할 수 있습니다.
*   **예약된 동시성 (Reserved Concurrency)**:
    *   특정 함수에 전용으로 할당할 동시 실행 수를 설정합니다. (예: 20개 설정 시, 해당 함수는 최대 20개까지 동시에 실행되며, 계정의 나머지 예약되지 않은 동시성 풀에서 20개가 차감됩니다.)
    *   예약된 동시성을 `0`으로 설정하고 함수를 호출하면, `Invocation überschreitet reservierte Concurrency` (호출이 예약된 동시성을 초과함) 또는 유사한 스로틀링 오류(429)가 발생합니다. 이를 통해 스로틀링 처리 로직을 테스트할 수 있습니다.
*   **프로비저닝된 동시성 (Provisioned Concurrency)**:
    *   콜드 스타트를 줄이기 위해 미리 초기화된 실행 환경(웜 풀)을 유지합니다.
    *   함수의 특정 버전(Version)이나 별칭(Alias)에 대해서만 설정 가능합니다 (`$LATEST` 버전에는 직접 설정 불가). 새 버전을 게시한 후 해당 버전에 프로비저닝된 동시성을 할당해야 합니다.
    *   프로비저닝된 동시성은 추가 비용이 발생하므로 필요한 만큼만 설정해야 합니다.

### Lambda 함수 종속성 관리

*   Lambda 함수가 외부 라이브러리(예: AWS X-Ray SDK, 데이터베이스 클라이언트)에 의존하는 경우, 해당 종속성을 코드와 함께 패키징하여 압축 파일(.zip)로 만들어 Lambda에 업로드해야 합니다.
*   **언어별 패키징 방법**:
    *   **Node.js**: `npm`을 사용하여 `node_modules` 디렉터리에 종속성을 설치합니다.
    *   **Python**: `pip install <library> -t <target_directory>`를 사용합니다.
    *   **Java**: 필요한 `.jar` 파일을 포함합니다.
*   **업로드 제한**:
    *   압축 파일 크기가 50MB 이하이면 Lambda 콘솔이나 CLI를 통해 직접 업로드할 수 있습니다.
    *   50MB를 초과하면 Amazon S3에 먼저 업로드한 후, Lambda 함수 생성/업데이트 시 해당 S3 위치를 참조해야 합니다.
*   **네이티브 라이브러리**: C/C++ 등으로 작성된 네이티브 라이브러리는 Lambda 실행 환경인 Amazon Linux에서 컴파일되어야 올바르게 작동합니다.
*   **AWS SDK**: 대부분의 AWS SDK(예: Node.js, Python, Java 용 SDK)는 Lambda 실행 환경에 기본적으로 포함되어 있으므로, 별도로 패키징할 필요가 없습니다. (단, X-Ray SDK와 같이 일부 SDK는 명시적으로 포함해야 할 수 있습니다.)

### 종속성을 가진 Lambda 함수 생성 실습 (Node.js, X-Ray SDK 예시)

1.  **CloudShell 환경 준비**: `mkdir lambda`로 작업 디렉터리 생성, `nano index.js`로 예제 코드 작성 ( `aws-xray-sdk-core` 요구, S3 `listBuckets` 호출).
2.  **종속성 설치**: `npm install aws-xray-sdk-core` 실행. `node_modules` 디렉터리와 `package-lock.json` 파일 생성됨.
3.  **파일 권한 수정 (필요시)**: `chmod` 명령 사용.
4.  **압축 파일 생성**: `zip functions.zip index.js node_modules/* package-lock.json -r` (또는 유사 명령)
5.  **IAM 역할 생성**:
    *   이름: `DemoLambdaWithDependencies`
    *   신뢰 관계: Lambda 서비스
    *   정책:
        *   `AWSLambdaBasicExecutionRole` (CloudWatch Logs 쓰기 권한)
        *   `AWSXRayDaemonWriteAccess` (X-Ray 추적 데이터 전송 권한, 나중에 활성화 시 자동 추가되기도 함)
        *   `AmazonS3ReadOnlyAccess` (S3 버킷 목록 조회 권한)
6.  **Lambda 함수 생성 (AWS CLI 사용)**:
    ```bash
    aws lambda create-function --function-name Lambda-xray-with-dependencies \
    --zip-file fileb://functions.zip --handler index.handler --runtime nodejs14.x \
    --role <복사한_IAM_역할_ARN>
    ```
7.  **X-Ray 활성화**: Lambda 함수 구성 > 모니터링 및 운영 도구 > 활성 추적 활성화.
8.  **테스트**:
    *   초기 테스트 시 S3 접근 권한 부족으로 `AccessDenied` 오류 발생 가능 (IAM 역할 전파 시간 소요 또는 권한 누락). 역할에 `AmazonS3ReadOnlyAccess` 정책이 올바르게 연결되었는지 확인.
    *   성공 시, S3 버킷 목록이 반환됨.
    *   **X-Ray 콘솔 확인**: [서비스 맵]에서 클라이언트 -> Lambda 함수 -> Amazon S3 호출 흐름 시각화. [추적]에서 각 호출의 성공/실패 여부, 지연 시간, 오류 발생 시 상세 정보(예: 403 AccessDenied) 확인 가능.

### CloudFormation을 이용한 Lambda 함수 배포

Lambda 함수는 AWS CloudFormation을 사용하여 인프라 코드로 관리하고 배포할 수 있습니다.

1.  **인라인 (Inline) 코드**:
    *   간단하고 종속성이 없는 Lambda 함수의 경우, CloudFormation 템플릿 내의 `Code.ZipFile` 속성에 직접 코드를 작성할 수 있습니다.
    *   종속성이 있는 함수에는 적합하지 않습니다.
2.  **S3를 통한 .zip 파일 (권장 방식)**:
    *   Lambda 함수 코드와 종속성을 .zip 파일로 패키징하여 Amazon S3 버킷에 업로드합니다.
    *   CloudFormation 템플릿에서는 `Code` 속성 하위에 `S3Bucket`, `S3Key`, (선택적으로) `S3ObjectVersion`을 지정하여 S3에 있는 .zip 파일을 참조합니다.
    *   S3 버킷 버저닝을 사용하는 경우, `S3ObjectVersion`을 업데이트하면 CloudFormation이 Lambda 함수를 자동으로 업데이트합니다. 이를 지정하지 않으면 S3의 파일을 덮어쓰더라도 CloudFormation은 변경을 감지하지 못할 수 있습니다.
3.  **다중 계정 배포**:
    *   소스 코드(.zip 파일)가 있는 S3 버킷(계정 A)과 다른 계정(계정 B, C)에 Lambda 함수를 배포해야 하는 경우:
        *   계정 A의 S3 버킷에는 계정 B, C의 CloudFormation 실행 역할이 해당 버킷의 객체를 읽을 수 있도록 허용하는 버킷 정책이 필요합니다.
        *   계정 B, C에서 CloudFormation 스택을 실행할 때 사용되는 실행 역할에는 계정 A의 S3 버킷에서 객체를 가져올 수 있는 권한이 있어야 합니다.

### CloudFormation을 이용한 Lambda 함수 배포 실습

1.  **S3 버킷 준비**:
    *   새 S3 버킷 생성 (예: `s3-cloudformation-lambda-demo-stephane`).
    *   **버킷 버저닝 활성화**.
    *   이전 실습에서 생성한 `functions.zip` 파일을 이 버킷에 업로드. 업로드된 객체의 **객체 버전 ID**를 기록.
2.  **CloudFormation 템플릿 (`lambda-xray.yaml`) 검토**:
    *   **매개변수 (Parameters)**: `S3Bucket`, `S3Key`, `S3ObjectVersion`을 입력받도록 정의.
    *   **리소스 (Resources)**:
        *   `LambdaExecutionRole` (IAM 역할): CloudWatch Logs 쓰기, X-Ray 추적 전송, S3 Get*/List* 작업을 허용하는 정책 포함.
        *   `LambdaWithXRay` (Lambda 함수):
            *   `Handler`: `index.handler`
            *   `Role`: 위에서 정의한 `LambdaExecutionRole`의 ARN (`!GetAtt LambdaExecutionRole.Arn`)
            *   `Code`: 매개변수로 받은 `S3Bucket`, `S3Key`, `S3ObjectVersion` 참조.
            *   `Runtime`: `nodejs14.x`
            *   `Timeout`: 10초
            *   `TracingConfig`: `Mode: Active` (X-Ray 활성화)
3.  **CloudFormation 스택 생성**:
    *   CloudFormation 콘솔에서 [스택 생성] > [준비된 템플릿] > [템플릿 파일 업로드] 선택 후 `lambda-xray.yaml` 파일 지정.
    *   스택 이름 입력 (예: `DemoLambdaCF`).
    *   매개변수 입력: S3 버킷 이름, `S3Key` (`functions.zip`), 기록해둔 `S3ObjectVersion`.
    *   IAM 리소스 생성 승인 후 스택 생성 시작.
4.  **확인**:
    *   스택 생성이 완료되면, Lambda 콘솔에서 CloudFormation에 의해 생성된 함수 확인. 함수 페이지에 CloudFormation으로 관리된다는 알림 표시.
    *   함수 테스트 실행 후, X-Ray 콘솔에서 추적 데이터 확인.
5.  **정리**: CloudFormation 콘솔에서 스택을 삭제하면 Lambda 함수와 IAM 역할 등 관련 리소스가 함께 삭제됨.

### Lambda 컨테이너 이미지 지원

*   Lambda 함수를 최대 10GB 크기의 컨테이너 이미지로 패키징하여 Amazon ECR(Elastic Container Registry)에서 배포할 수 있습니다.
*   이를 통해 복잡하거나 큰 종속성을 가진 애플리케이션, 또는 기존 컨테이너 기반 워크플로우를 Lambda와 통합하기 용이합니다.
*   **핵심 요구사항**: 사용되는 베이스 이미지는 **Lambda 런타임 API**를 구현해야 합니다.
    *   AWS는 주요 언어(Python, Node.js, Java, .NET, Go, Ruby)에 대해 Lambda 런타임 API가 구현된 베이스 이미지를 제공합니다.
    *   사용자가 직접 커스텀 런타임을 위한 베이스 이미지를 만들 수도 있습니다.
*   **로컬 테스트**: Lambda 런타임 인터페이스 에뮬레이터(RIE)를 사용하여 로컬 환경에서 컨테이너 이미지를 테스트할 수 있습니다.
*   **배포 흐름**: Dockerfile 작성 -> 이미지 빌드 -> ECR에 푸시 -> Lambda 함수 생성 시 ECR 이미지 URI 지정.

### Lambda 컨테이너 이미지 Dockerfile 예시 및 최적화

*   **Dockerfile 예시 (Node.js)**:
    ```dockerfile
    FROM amazon/aws-lambda-nodejs:12 # AWS 제공 Node.js 12 베이스 이미지 사용
    COPY app.js package*.json ./    # 애플리케이션 코드 및 package.json 복사
    RUN npm install                 # 종속성 설치
    CMD ["app.lambdaHandler"]       # Lambda 호출 시 실행될 핸들러 지정
    ```
*   **컨테이너 이미지 최적화 방안**:
    1.  **AWS 제공 베이스 이미지 사용**: Amazon Linux 2 기반의 AWS 제공 이미지는 Lambda 서비스에 의해 캐시될 수 있어 초기화 속도에 유리합니다.
    2.  **멀티 스테이지 빌드 활용**: 빌드 과정에서만 필요한 도구나 라이브러리를 최종 이미지에서 제외하여 이미지 크기를 최소화합니다.
    3.  **계층(Layer) 순서 최적화**: Dockerfile에서 자주 변경되지 않는 부분(예: 기본 패키지 설치)을 앞쪽에, 자주 변경되는 애플리케이션 코드 복사 등을 뒤쪽에 배치하여 빌드 캐시를 효율적으로 활용합니다.
    4.  **큰 계층(Layer)을 가진 함수는 단일 리포지토리 사용**: ECR에서 계층을 비교하고 중복 저장을 피하는 데 도움이 될 수 있습니다.
*   **주요 사용 사례**: 10GB에 가까운 매우 큰 애플리케이션(코드+종속성)을 Lambda로 배포해야 할 경우 유용합니다.


## Lambda 버전, 별칭, CodeDeploy, 함수 URL, 제한 및 모범 사례 요약

### 1. Lambda 버전 관리

*   **`$LATEST` 버전**: 개발 중인 최신 버전으로, 코드 및 설정 수정이 가능합니다.
*   **게시된 버전 (예: V1, V2, ...)**:
    *   특정 시점의 코드와 설정을 스냅샷으로 만든 것으로, **변경이 불가능(immutable)**합니다.
    *   새로운 버전은 이전 버전을 기반으로 게시되며, 버전 번호가 증가합니다.
    *   각 버전은 고유한 ARN(Amazon Resource Name)을 가지며, 독립적으로 존재합니다.
    *   이를 통해 안정적인 릴리스 관리와 롤백이 용이해집니다.

### 2. Lambda 별칭 (Alias)

*   **정의**: Lambda 함수의 특정 버전을 가리키는 **포인터**입니다. (예: `dev`, `test`, `prod`)
*   **특징**:
    *   별칭 자체는 변경 가능하며, 가리키는 함수 버전을 쉽게 업데이트할 수 있습니다.
    *   각 별칭은 고유한 ARN을 가집니다.
    *   **중요**: 별칭은 다른 별칭을 참조할 수 없으며, 오직 Lambda 함수의 특정 버전만 참조할 수 있습니다. (시험 출제 포인트)
*   **활용**:
    *   **안정적인 엔드포인트 제공**: 최종 사용자나 다른 서비스는 특정 별칭 ARN을 호출하며, 백엔드에서는 해당 별칭이 가리키는 버전을 변경하여 업데이트를 배포할 수 있습니다.
    *   **블루/그린 배포**: 별칭의 트래픽 가중치 설정을 통해 점진적인 배포가 가능합니다.
        *   예: `prod` 별칭이 V1을 100% 가리키다가, V2를 10%, V1을 90% 가리키도록 변경하여 V2의 안정성을 테스트한 후, 점차 V2의 비중을 늘려 최종적으로 100% V2로 전환할 수 있습니다.

### 3. Lambda와 CodeDeploy 통합

*   CodeDeploy를 사용하면 Lambda 별칭의 트래픽 전환을 자동화하여 안전한 배포를 수행할 수 있습니다.
*   **배포 전략**:
    *   **선형(Linear)**: 예) `Linear10PercentEvery3Minutes` (3분마다 10%씩 트래픽 증분).
    *   **카나리(Canary)**: 예) `Canary10Percent5Minutes` (5분 동안 10% 트래픽을 새 버전으로 보내 테스트 후, 문제 없으면 나머지 90%도 전환).
    *   **한 번에 모두(All-at-once)**: 즉시 모든 트래픽을 새 버전으로 전환 (가장 빠르지만 위험).
*   **롤백**: 배포 중 문제 발생 시(예: CloudWatch 경보 발생, 후크 실패), CodeDeploy가 자동으로 이전 버전으로 롤백합니다.
*   **`AppSpec.yml` 파일**: CodeDeploy 배포 구성을 정의하는 파일로, 다음 항목이 중요합니다.
    *   `Name`: 배포 대상 Lambda 함수 이름.
    *   `Alias`: 트래픽을 이전할 Lambda 함수 별칭 이름.
    *   `CurrentVersion`: 현재 트래픽을 받고 있는 Lambda 함수 버전.
    *   `TargetVersion`: 새로 트래픽을 받을 Lambda 함수 버전.

### 4. Lambda 함수 URL

*   API Gateway나 Application Load Balancer(ALB)를 설정하는 번거로움 없이, Lambda 함수에 **직접 HTTP(S) 엔드포인트**를 부여하는 기능입니다.
*   **특징**:
    *   함수 또는 별칭마다 고유하고 변경되지 않는 URL이 생성됩니다.
    *   IPv4 및 IPv6를 모두 지원합니다.
    *   웹 브라우저, `curl`, Postman 등 모든 HTTP 클라이언트에서 호출 가능합니다.
    *   **주의**: 함수 URL은 퍼블릭 인터넷을 통해서만 액세스 가능하며, VPC 내 프라이빗 액세스는 지원하지 않습니다.
    *   다른 도메인에서의 호출을 위해 CORS(Cross-Origin Resource Sharing) 구성이 가능합니다.
*   **보안**:
    *   리소스 기반 정책을 사용하여 함수 URL에 대한 접근을 제어합니다.
    *   **인증 유형(AuthType)**:
        *   `NONE`: 인증 없이 공개적으로 접근 가능합니다. 이 경우, 리소스 기반 정책에서 `Principal: "*"` 와 같이 설정하여 모든 사용자에게 `lambda:InvokeFunctionUrl` 권한을 부여해야 합니다.
        *   `AWS_IAM`: IAM을 사용하여 요청을 인증하고 권한을 부여합니다.
            *   **동일 계정**: 호출자의 자격 증명 기반 정책 또는 Lambda 함수의 리소스 기반 정책 중 하나에서 허용되면 됩니다.
            *   **교차 계정**: 호출자의 자격 증명 기반 정책 **및** Lambda 함수의 리소스 기반 정책 **모두**에서 허용되어야 합니다.
    *   함수 URL은 특정 버전이 아닌, `$LATEST` 버전이나 특정 별칭에만 적용할 수 있습니다.
*   **스로틀링**: 예약된 동시성 설정을 통해 함수 URL을 통한 호출량을 제어할 수 있습니다.

### 5. Lambda 제한 사항 (시험 대비)

Lambda에는 실행 및 배포와 관련된 몇 가지 중요한 제한이 있으며, 이는 지역별로 다를 수 있습니다.

*   **실행 제한**:
    *   **메모리 할당**: 128MB ~ 10GB (1MB 단위로 조절 가능). 메모리 증가는 vCPU 성능 향상으로 이어집니다.
    *   **최대 실행 시간 (Timeout)**: 900초 (15분). 이 시간을 초과하는 작업은 Lambda에 부적합합니다.
    *   **환경 변수 크기**: 최대 4KB.
    *   **임시 디스크 공간 (`/tmp` 디렉터리)**: 512MB.
    *   **동시 실행 수**: 계정당 기본 1,000개. (AWS 지원팀을 통해 상향 조정 가능).
*   **배포 제한**:
    *   **압축된 배포 패키지 크기 (.zip 파일)**: 50MB.
    *   **압축 해제된 배포 패키지 크기**: 250MB. (더 큰 파일은 `/tmp` 공간 활용 고려).
    *   **환경 변수 크기**: 최대 4KB.
    *   **계층 (Layers)**:
        * 함수당 최대 5개의 계층 사용 가능
        * 모든 계층의 압축 해제된 총 크기는 250MB를 초과할 수 없음
        * 각 계층의 크기는 50MB를 초과할 수 없음
    *   **컨테이너 이미지**:
        * ECR에서 호스팅되는 컨테이너 이미지의 최대 크기: 10GB
        * 컨테이너 이미지는 Lambda 런타임 API를 구현해야 함
    *   **함수 버전**:
        * 함수당 최대 $LATEST를 포함하여 무제한 버전 생성 가능
        * 각 버전은 고유한 ARN을 가짐
        * 게시된 버전은 변경 불가능 (immutable)
    *   **별칭 (Alias)**:
        * 함수당 최대 100개의 별칭 생성 가능
        * 별칭은 특정 함수 버전을 가리키며, 다른 별칭을 참조할 수 없음
        * 트래픽 분할을 통한 가중치 기반 라우팅 지원
    *   **배포 패키지 업로드**:
        * 직접 업로드: 50MB 이하
        * S3를 통한 업로드: 250MB 이하
        * 컨테이너 이미지: 10GB 이하
    *   **배포 구성**:
        * IAM 역할과 정책 크기: IAM 제한에 따름
        * VPC 구성: 함수당 최대 16개의 서브넷과 보안 그룹 지정 가능
        * 리소스 기반 정책: 최대 20KB

이러한 제한을 초과하는 요구사항이 있다면, 다음과 같은 대안을 고려해볼 수 있습니다:

1. **대용량 파일 처리**:
   * S3를 임시 저장소로 활용
   * `/tmp` 디렉토리 (최대 10GB)를 활용한 청크 단위 처리
   * 여러 Lambda 함수로 분할하여 처리

2. **컨테이너 기반 워크로드**:
   * Amazon ECS (Elastic Container Service)
   * Amazon EKS (Elastic Kubernetes Service)
   * AWS Fargate

3. **장시간 실행 워크로드**:
   * AWS Batch
   * Amazon EC2
   * AWS Step Functions를 통한 워크플로우 조정

4. **대규모 메모리 요구사항**:
   * Amazon EC2 메모리 최적화 인스턴스
   * Amazon ElastiCache
   * Amazon RDS

### 6. Lambda 모범 사례

#### 1. 성능 최적화
- **실행 컨텍스트 재사용**: 
  - 데이터베이스 연결, AWS SDK 클라이언트 초기화, 대용량 종속성 로드 등은 **핸들러 함수 외부 (글로벌 스코프)**에서 수행
  - 실행 컨텍스트가 재사용될 때 시간을 절약 (콜드 스타트 이후 웜 스타트 시 성능 향상)

#### 2. 구성 관리
- **환경 변수 활용**: 
  - 코드 내 하드코딩 대신 환경 변수를 사용하여 데이터베이스 연결 문자열, S3 버킷 이름 등 변경 가능한 설정을 관리
  - 운영 매개변수를 함수에 전달하는 데 활용
- **민감 정보 보안**: 
  - 비밀번호나 API 키와 같은 민감한 정보는 AWS KMS를 사용하여 암호화 후 환경 변수로 저장

#### 3. 코드 및 패키지 관리
- **핸들러 분리**: 
  - 핵심 로직에서 Lambda 핸들러(진입점)를 분리
- **패키지 최적화**:
  - 런타임에 필요한 코드와 종속성만 포함하도록 패키지 크기를 최소화
  - Lambda가 배포 패키지를 압축 해제하는 데 걸리는 시간을 최소화
  - 함수가 너무 크면 분리 고려
- **Lambda 계층 활용**: 
  - 여러 함수에서 공통으로 사용되는 라이브러리나 종속성은 계층으로 분리
  - 재사용성을 높이고 배포 패키지 크기를 줄임

#### 4. 주의사항
- **재귀 호출 방지**: 
  - Lambda 함수가 자기 자신을 직접 또는 간접적으로 계속 호출하는 재귀적 패턴은 피해야 함
  - 의도치 않은 비용 증가와 스로틀링을 유발할 수 있음
- **종속성 관리**: 
  - 종속성의 복잡성을 최소화
  - 함수 배포 패키지의 종속성을 제어

이러한 모범 사례들을 따르면 Lambda 함수의 성능을 향상시키고, 유지보수성을 높이며, 비용을 최적화할 수 있습니다.

AWS Lambda 함수를 사용할 때의 모범 사례는 다음과 같습니다.

 – 핵심 로직에서 Lambda 핸들러(진입점)를 분리합니다.

 – 실행 컨텍스트 재사용을 활용하여 함수의 성능을 향상시키세요.

 – AWS Lambda 환경 변수를 사용하여 함수에 운영 매개변수를 전달합니다.

 – 함수 배포 패키지의 종속성을 제어합니다.

 – 배포 패키지 크기를 런타임 요구 사항에 맞게 최소화합니다.

 – Lambda가 배포 패키지를 압축 해제하는 데 걸리는 시간을 줄입니다.

 – 종속성의 복잡성을 최소화합니다.

 – 재귀 코드 사용을 피하세요

따라서 이 시나리오에서 정답은 다음과 같습니다.

– 실행 컨텍스트 재사용을 활용하여 함수의 성능을 향상시키세요.

– AWS Lambda 환경 변수를 사용하여 함수에 운영 매개변수를 전달합니다. 