# AWS 기타 서비스 정리

## 1. Amazon SES (Simple Email Service)
- AWS에서 가장 단순한 이메일 서비스
- 주요 기능:
  - 이메일 발신/수신 가능
  - SMTP 인터페이스 사용 가능
  - AWS SDK 지원
  - S3, SNS, Lambda 등과 통합 가능
  - IAM과 완전 통합
- 시험 관련 팁:
  - 이메일 관련 문제가 나오면 SES가 정답일 가능성이 높음
  - 이메일 기능 외에는 다른 용도로 사용하지 않음

## 2. Amazon OpenSearch Service
- 이전의 Amazon ElasticSearch의 후속작 (라이선싱 문제로 이름 변경)
- 주요 특징:
  - 모든 필드에서 부분 매칭 검색 가능
  - 분석적 쿼리 지원
  - 관리형/서버리스 클러스터 옵션 제공
  - SQL 플러그인으로 SQL 호환성 지원
  - OpenSearch 대시보드로 시각화 가능
- 데이터 소스:
  - Kinesis Data Firehose
  - IoT
  - CloudWatch Logs
  - 커스텀 앱
- 보안 기능:
  - Cognito, IAM 통합
  - 저장 및 전송 중 암호화
- 일반적인 사용 패턴:
  - DynamoDB와 연동하여 검색 기능 구현
    - DynamoDB Stream → Lambda → OpenSearch
    - 부분 검색으로 항목 ID 찾기 → DynamoDB에서 전체 항목 조회
  - CloudWatch Logs 데이터 분석
    - 구독 필터 → Lambda → OpenSearch (실시간)
    - 구독 필터 → Kinesis Data Firehose → OpenSearch (근실시간)
  - Kinesis Data Streams를 통한 실시간 데이터 처리
    - Kinesis Data Streams → Kinesis Data Firehose → OpenSearch
    - Kinesis Data Streams → Lambda → OpenSearch
- DynamoDB와의 차이점:
  - DynamoDB: 인덱스/기본키로만 쿼리 가능
  - OpenSearch: 모든 필드에서 부분 매칭 검색 가능

## 3. Amazon Athena
- S3 버킷의 데이터를 위한 서버리스 SQL 쿼리 서비스
- 주요 특징:
  - Presto 엔진 기반
  - 서버리스 아키텍처
  - 데이터 이동 없이 S3에서 직접 분석
  - 다양한 파일 포맷 지원 (CSV, JSON, ORC, Avro, Parquet)
  - TB당 고정 비용 과금
  - QuickSight와 통합하여 보고서/대시보드 생성
- 주요 사용 사례:
  - 임의 쿼리 실행
  - 비즈니스 인텔리전스/분석
  - AWS 서비스 로그 분석 (VPC 흐름 로그, 로드밸런서 로그, CloudTrail 등)
- 성능 최적화:
  - Columnar 데이터 포맷 사용 (Parquet, ORC)
    - 필요한 열만 스캔하여 비용 절감
    - Glue를 사용하여 CSV → Parquet 변환 가능
  - 데이터 압축
  - 데이터 파티셔닝
    - S3 경로에 파티션 정보 포함 (예: /year=1991/month=1/day=1)
    - 필요한 파티션만 스캔하여 성능 향상
  - 큰 파일 사용 (128MB 이상)
    - 작은 파일보다 스캔/반환이 용이
- 연합 쿼리 기능:
  - Lambda를 통한 외부 데이터 소스 쿼리
  - 지원 데이터베이스:
    - ElastiCache
    - DocumentDB
    - DynamoDB
    - Redshift
    - Aurora
    - SQL Server
    - MySQL
    - EMR 기반 HBase
    - 온프레미스 데이터베이스
  - 쿼리 결과는 S3에 저장
- 시험 팁:
  - S3에서 서버리스 SQL 분석이 필요한 경우 Athena가 정답일 가능성 높음

## 4. Amazon MSK (Managed Streaming for Apache Kafka)
- Apache Kafka를 위한 완전관리형 서비스
- 특징:
  - Kinesis와 유사한 데이터 스트리밍 서비스
  - 관리형/서버리스 옵션 제공
  - VPC 내 다중 AZ 배포 가능
  - 자동 복구 기능
- Kinesis와의 차이점:
  - 메시지 크기: MSK는 10MB까지 확장 가능 (Kinesis는 1MB 제한)
  - 확장성: MSK는 파티션 추가만 가능
  - 암호화: MSK는 플레인텍스트/TLS 선택 가능
- 데이터 처리 옵션:
  - Kinesis Data Analytics (Apache Flink)
  - AWS Glue (Apache Spark Streaming)
  - Lambda
  - 커스텀 Kafka 컨슈머

## 5. AWS Certificate Manager (ACM)
- SSL/TLS 인증서 관리 서비스
- 주요 기능:
  - 인증서 프로비저닝, 관리, 배포
  - 공용/사설 인증서 지원
  - 무료 공용 인증서
  - 자동 갱신
- 통합 서비스:
  - Elastic Load Balancer
  - CloudFront
  - API Gateway
- Private CA 기능:
  - 사설 인증서 발급
  - 내부 TLS 통신용
  - AWS 조직 내부 사용

## 6. Amazon Macie
- 데이터 보안 및 프라이버시 서비스
- 주요 기능:
  - 머신러닝/패턴 매칭 기반 민감 정보 탐지
  - PII(개인식별정보) 보호
  - S3 버킷 분석
  - EventBridge 통합
  - SNS/Lambda 연동

## 7. AWS AppConfig
- 애플리케이션 구성 관리 서비스
- 주요 기능:
  - 동적 구성 데이터 관리
  - 기능 플래그 지원
  - 점진적 배포
  - 구성 검증
- 구성 소스:
  - Parameter Store
  - SSM 문서
  - S3 버킷
- 모니터링:
  - CloudWatch 통합
  - 자동 롤백

## 8. CloudWatch Evidently
- 새로운 기능 테스트 서비스
- 주요 기능:
  - Launches (기능 플래그)
    - 일부 사용자에게만 새로운 기능 활성화/비활성화
    - 점진적인 기능 출시 관리 (예: 사용자의 5%에게만 제공)
    - 위험 감소를 위한 단계적 출시
  - Experiments (A/B 테스팅)
    - 다양한 기능 버전 비교 테스트
    - 사용자 반응 및 성능 측정
    - UI 요소 위치 등 다양한 변수 테스트
  - Overrides (특정 사용자 대상 테스트)
    - 베타 테스터 등 특정 사용자 그룹에 기능 강제 적용
    - 사용자 ID 기반 기능 제어
    - 무작위 할당 방지
- 구현 방식:
  - 코드 스니펫을 앱에 내장
  - 사용자 비율 설정으로 기능 노출 제어
  - 실험 결과 모니터링 및 분석
- 데이터 저장:
  - CloudWatch Logs
  - Amazon S3
- 활용 사례:
  - 새로운 기능의 안전한 출시
  - 사용자 경험 최적화
  - 기능 성능 모니터링
- 시험 팁:
  - 기능의 점진적 출시나 A/B 테스팅 관련 문제에서 주요 출제 포인트
