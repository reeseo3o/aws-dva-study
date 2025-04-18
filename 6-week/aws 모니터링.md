## AWS의 모니터링 개요
| 레이어 | 도구 | 주 기능 | 언제 사용? |
|--------|------|---------|-----------|
| **인프라/시스템** | **CloudWatch Metrics & Alarms** | CPU·네트워크·디스크 I/O, Auto Scaling, 경보 | 리소스 상태·스케일 자동화 |
| **로그/이벤트** | **CloudWatch Logs, EventBridge** | 로그 수집·검색, 이벤트 버스 | 보안·실시간 WorkFlow |
| **트레이싱** | **AWS X‑Ray, OpenTelemetry** | 서비스 간 호출 지연, 분산 추적 | 마이크로서비스 병목 분석 |
| **감사** | **CloudTrail** | API 호출 기록 | 규정 준수, 보안 포렌식 |
| **가용성 테스트** | **CloudWatch Synthetics** | 웹 CANARY, API 모니터링 | 외부 사용자 관점 가용성 |

---

## CloudWatch 지표
* **Namespace / Metric / Dimensions(≤ 10)**  
* EC2 기본 5 분, **Detailed = 1 분**(추가 과금)  
* **Metric Math** : 지표끼리 계산 → 새 그래프/경보  
* **보존** : 15 개월(해상도별 롤업)  



# CloudWatch 사용자 지정 지표

## 항목
| 항목                 | 설명                                                       |
|---------------------|----------------------------------------------------------|
| PutMetricData API/CLI | 값·단위·타임스탬프(과거 2주 ~ 미래 2h) 업로드                    |
| StorageResolution    | 60 s(표준) / 1 s(고해상도)                                    |
| Dimensions(≤ 30)     | EC2 한계(10)보다 확장, 태그형 세분화                           |
| Agent 자동 수집     | Unified CloudWatch Agent → RAM, Disk, Swap 자동 전송            |

## CloudWatch 로그
- **Log Group ↔ Log Stream 구조**

### Retention
1일 ~ 10년 또는 무제한

### 암호화
기본 + 선택적 KMS CMK

### Logs Insights
SQL‑like 쿼리 + 실시간 그래프 + 대시보드 고정

## CloudWatch 로그 실습
### Agent 설치 (Linux)

```bash
sudo yum install -y awslogs
sudo vim /etc/awslogs/awslogs.conf   # 수집 파일·그룹 설정
sudo systemctl start awslogsd
```
### Logs Insights 쿼리

```sql
filter @message like /ERROR/
| stats count() by bin(1m)
| sort @timestamp desc
```
### Export to S3 (배치)

```bash
aws logs create-export-task --log-group-name /prod/app \
     --from 1713600000 --to 1713686400 \
     --destination s3://my-logs --destination-prefix prod/app/
```
### Subscription Filter (실시간)

```bash
aws logs put-subscription-filter \
     --log-group /prod/app --filter-name error-only \
     --filter-pattern '"ERROR"' \
     --destination-arn arn:aws:firehose:ap-northeast-2:123456789012:deliver-err
```

## CloudWatch 에이전트 및 CloudWatch Logs 에이전트

| Agent                    | 주요 기능                               | 권장 상황                                   |
|--------------------------|---------------------------------------|-------------------------------------------|
| CloudWatch Logs Agent    | 파일·Syslog를 Log Group으로 전송        | 경량 EC2, 온프레미스                       |
| Unified CloudWatch Agent | Metrics + Logs 한 번에 수집, SSM 관리 | RAM, Disk % 등 필요, 대규모               |
| Fluent Bit (FireLens)    | 컨테이너 로그 라우팅                     | ECS/EKS                                    |

## CloudWatch Logs - 메트릭 필터
로그 패턴 → 실시간 지표 생성

예) Nginx status 5xx 카운트

```bash
aws logs put-metric-filter \
  --log-group-name /prod/app \
  --filter-name "5xxCount" \
  --metric-transformations \
      metricName=HTTP5xx,metricNamespace=Custom/Web,metricValue=1 \
  --filter-pattern '" 5"'
```
이후 CloudWatch Alarm으로 정상 지표처럼 사용.

## CloudWatch 경보

| 타입            | 설명                               |
|---------------|------------------------------------|
| Static Threshold | 단순 임계값(> 80%)                   |
| Anomaly Detection | 과거 패턴 기반 자동 밴드             |
| Composite Alarm | AND/OR 조합, 잡음 감소              |
| Actions         | SNS, Auto Scaling, EC2 Recovery, Systems Manager OpsItem |

## CloudWatch Synthetics
Canary : Headless Chrome/Puppeteer 스크립트로 가용성·레이턴시 모니터링

- 스케줄(1 분 ~ 1 시간)
- 결과 S3 저장 + CloudWatch Alarm 연동
- 접근 제한 사이트 → VPC 엔드포인트 지정 가능

## Amazon EventBridge
Serverless 이벤트 버스 – CloudWatch Events의 진화판

- **소스**: SaaS 앱, 자체 App(사용자), AWS 서비스 이벤트
- **루트**: 규칙 → 타겟(Lambda, Step Functions, SNS, API Destinations 등)
- **스케줄러**: at() / cron() 표현식 지원
- **리플레이**: 최근 24 시간 이벤트 재전송

## Amazon EventBridge - 다중 계정 통합
- EventBus Policy 로 SourceAccount or SourceArn 허용
- EventBridge Organization 진입점 (모든 계정 이벤트 일괄 수신)
- 글로벌: 이벤트를 다른 리전·계정 버스로 수 초 내 복제 (비동기)

## AWS X-Ray
### X-Ray 개요
분산 추적 서비스 – HTTP · Lambda · SDK 호출 경로 시각화

- **개념**: Segment(서비스 단위) + Subsegment(쿼리, 다운스트림) → Trace(요청 전체)

### X-Ray: 계측 및 개념

| 환경             | 방법                                                   |
|-----------------|--------------------------------------------------------|
| Lambda          | 콘솔 체크박스 + AWSXRayTracingState 헤더                   |
| SDK(AWS, HTTP) | X-Ray SDK(Node, Java, Python, Go, .NET) / OpenTelemetry SDK |
| 자동 계측        | ECS Fargate → X-Ray Daemon Sidecar                      |

### X-Ray: 샘플링 규칙
- Centralized rules (JSON) : rate, reservoir, service_name, http_method, url_path
- 기본 1 req/s + 5%

### X-Ray: API
- PutTraceSegments, PutTelemetryRecords, GetServiceGraph— 커스텀 솔루션에서 호출 가능

### Beanstalk를 사용한 X-Ray
- EB 콘솔 → "X-Ray 활성화" → 플랫폼이 데몬 설치, 환경 변수 세팅

### X-Ray 및 ECS
- Daemon Sidecar (bridge network) + Task IAM xray:PutTraceSegments

### OpenTelemetry를 위한 AWS Distro
- ADOT Collector : OTLP → X-Ray, CloudWatch, Prometheus, …
- EKS 용 Helm 차트, Lambda Layer 제공

## CloudTrail
- 모든 AWS API 호출(콘솔·SDK·CLI) 로그 → S3 (+ CloudWatch Logs 선택)
- **Insight Events**: 비정상 API 패턴 감지
- **Lake**: SQL-like 쿼리, 이벤트 보존·분석 단순화

### CloudTrail - EventBridge 통합
- CloudTrail → 이벤트 유형(WriteOnly, Insight) 선택
- EventBridge 규칙으로 실시간 SNS/Lambda 타깃

```json
{
  "source": ["aws.cloudtrail"],
  "detail-type": ["AWS API Call via CloudTrail"],
  "detail": { "eventSource": ["ec2.amazonaws.com"], "eventName": ["TerminateInstances"] }
}
```

## CloudTrail 대 CloudWatch 대 X-Ray

| 테이블        | CloudTrail        | CloudWatch       | X-Ray          |
|---------------|-------------------|------------------|----------------|
| 주요 목적     | 감사/보안         | 모니터링/알람    | 분산 추적       |
| 데이터 종류   | API Event         | Metric·Log       | Trace Segment  |
| 지연          | 수 분 ~ 15 분     | 실시간(지표), 초(로그) | 실시간        |
| 저장 위치     | S3, Lake          | CloudWatch DB    | X-Ray DB       |
| 주요 소비자   | 보안팀, 규정 감사 | 운영·SRE        | 개발·성능팀    |


