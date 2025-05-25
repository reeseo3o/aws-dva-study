*   **DynamoDB 기본**
    *   완전 관리형 NoSQL 데이터베이스 서비스로, 어떤 규모에서도 일관된 한 자릿수 밀리초 성능을 제공합니다.
    *   키-값 및 문서 데이터 모델을 지원하며, 관계형 데이터베이스와 달리 스키마가 유연합니다.
    *   테이블, 항목(행), 속성(열)으로 구성되며, 각 항목은 최대 400KB까지 저장 가능합니다.
    *   지원하는 데이터 타입:
        *   스칼라 타입: 숫자, 문자열, 이진수, Boolean, Null
        *   문서 타입: List, Map
        *   집합 타입: 문자열 집합, 숫자 집합, 이진수 집합
    *   파티션 키(해시 키)는 데이터 분산을 결정하며, 정렬 키(범위 키)는 파티션 내 데이터 정렬에 사용됩니다.

*   **읽기/쓰기 용량 모드**
    *   **프로비저닝 모드 (Provisioned Mode)**
        *   미리 정의된 읽기/쓰기 용량을 설정하고 비용 지불
        *   WCU (Write Capacity Units):
            *   1KB 이하 항목당 1 WCU 필요
            *   1KB 초과 시 올림하여 계산 (예: 2.5KB = 3 WCU)
            *   트랜잭션 쓰기는 2배의 WCU 소비
        *   RCU (Read Capacity Units):
            *   최종적 일관된 읽기: 4KB당 0.5 RCU
            *   강력한 일관된 읽기: 4KB당 1 RCU
            *   트랜잭션 읽기는 2배의 RCU 소비
        *   Auto Scaling 기능으로 자동 용량 조정 가능
    *   **온디맨드 모드 (On-Demand Mode)**
        *   사용량에 따라 자동으로 확장/축소
        *   초당 최대 40,000 RCU 및 WCU까지 즉시 수용
        *   요청당 비용 지불 방식으로, 예측이 어려운 워크로드에 적합
        *   프로비저닝 모드보다 비용이 높을 수 있음

*   **읽기 일관성**
    *   **최종적 일관된 읽기 (Eventually Consistent Read)**
        *   기본 설정값
        *   최근 완료된 쓰기가 즉시 반영되지 않을 수 있음
        *   1초 이내에 모든 복제본이 일관성을 가짐
    *   **강력한 일관된 읽기 (Strongly Consistent Read)**
        *   가장 최근에 완료된 쓰기가 즉시 반영
        *   더 많은 RCU를 소비하고 지연 시간이 증가할 수 있음
        *   글로벌 테이블에서는 사용 불가

*   **API 및 주요 작업**
    *   **데이터 쓰기**
        *   `PutItem`: 새 항목 생성 또는 전체 교체
        *   `UpdateItem`: 특정 속성 수정 또는 새 속성 추가
            *   원자적 카운터 구현 가능
            *   조건부 업데이트 지원
        *   `DeleteItem`: 단일 항목 삭제
    *   **데이터 읽기**
        *   `GetItem`: 기본 키로 단일 항목 조회
        *   `Query`: 
            *   파티션 키 값 필수
            *   정렬 키 조건 사용 가능 (>, <, Between, Begins with 등)
            *   인덱스 사용 가능
            *   페이지네이션 지원
        *   `Scan`: 
            *   전체 테이블 스캔 (비효율적)
            *   병렬 스캔 가능
            *   필터 표현식으로 결과 제한 가능
    *   **일괄 작업**
        *   `BatchWriteItem`: 최대 25개 Put/Delete 작업
        *   `BatchGetItem`: 
            *   최대 100개 항목 조회
            *   한 번에 가져올 수 있는 최대 데이터: 16MB 이하
            *   UnprocessedKeys 발생 가능한 상황:
                *   총 응답 크기가 16MB를 초과
                *   RCU(읽기 처리량) 초과
                *   파티션당 1MB 초과
                *   내부 오류 발생
            *   UnprocessedKeys 처리 전략:
                *   UnprocessedKeys를 사용한 재시도 필요
                *   즉시 재시도는 지양 (서버 과부하 상태 지속 가능)
                *   지수 백오프(Exponential Backoff) 적용
                    *   재시도 간격 점진적 증가 (100ms, 200ms, 400ms, ...)
                    *   무작위 지연(jitter) 추가로 서버 혼잡 방지
                *   AWS SDK 활용 권장
                    *   내장된 지수 백오프 기능 제공
                    *   UnprocessedKeys 자동 처리 지원
        *   실패한 작업에 대한 재시도 로직 필요

*   **인덱스**
    *   **로컬 보조 인덱스 (LSI)**
        *   테이블 생성 시에만 정의 가능
        *   파티션 키는 기본 테이블과 동일
        *   다른 정렬 키 사용
        *   강력한 일관된 읽기 지원
        *   테이블당 최대 5개 생성 가능
    *   **글로벌 보조 인덱스 (GSI)**
        *   언제든지 생성/삭제 가능
        *   다른 파티션 키와 정렬 키 사용 가능
        *   최종적 일관된 읽기만 지원
        *   테이블당 최대 20개 생성 가능
        *   별도의 RCU/WCU 설정 필요
        *   **용량 관리 주요 포인트**
            *   **WCU (Write Capacity Units) 관리**
                *   GSI의 WCU는 반드시 기본 테이블의 WCU보다 크거나 같아야 함
                *   기본 테이블 쓰기 작업 시 GSI도 자동 업데이트되어야 하기 때문
                *   GSI의 WCU가 부족하면 기본 테이블의 쓰기 작업도 제한(throttle)됨
            *   **RCU (Read Capacity Units) 관리**
                *   GSI의 RCU는 기본 테이블의 RCU와 독립적으로 설정 가능
                *   GSI에 대한 쿼리는 GSI의 RCU를 소비
                *   기본 테이블의 RCU와 GSI의 RCU는 서로 영향을 주지 않음
            *   **Auto Scaling 설정**
                *   GSI의 WCU/RCU에 대해 자동 확장/축소 설정 가능
                *   목표 사용률(예: 70%)을 기준으로 용량 자동 조정
                *   최소/최대 용량 범위 설정 필요
            *   **모니터링 및 경고**
                *   CloudWatch를 통한 용량 사용량 모니터링
                *   제한(throttling) 이벤트에 대한 경보 설정
                *   정기적인 용량 사용량 분석 수행
            *   **모범 사례**
                *   GSI 생성 시 초기 WCU를 기본 테이블의 WCU보다 약간 높게 설정
                *   Auto Scaling 설정으로 자동 용량 조정
                *   CloudWatch 경보를 설정하여 제한(throttling) 모니터링
                *   정기적인 용량 사용량 분석 수행
                *   비용 최적화를 위해 불필요한 GSI 제거

*   **DynamoDB Streams**
    *   테이블의 데이터 수정 이벤트를 시간 순서대로 기록
    *   최대 24시간 동안 데이터 보존
    *   네 가지 보기 유형:
        *   KEYS_ONLY: 수정된 항목의 키만 기록
        *   NEW_IMAGE: 수정 후의 전체 항목 이미지
        *   OLD_IMAGE: 수정 전의 전체 항목 이미지
        *   NEW_AND_OLD_IMAGES: 수정 전후의 항목 이미지
    *   Lambda 함수와 통합하여 이벤트 기반 아키텍처 구현
    *   Kinesis Data Streams로 데이터 내보내기 가능

*   **DynamoDB Streams 모니터링 및 로깅**
    *   **CloudWatch 모니터링**
        *   주요 메트릭스:
            *   `SuccessfulRequestLatency`: 성공한 요청의 지연 시간
            *   `ThrottledRequests`: 제한된 요청 수
            *   `SystemErrors`: 서버 측 오류
            *   `UserErrors`: 클라이언트 측 오류
            *   `ConsumedReadCapacityUnits`: 소비된 읽기 용량
            *   `ConsumedWriteCapacityUnits`: 소비된 쓰기 용량
    
    *   **CloudTrail 로깅**
        *   DynamoDB API 호출 상세 정보 기록:
            *   호출자의 IAM 사용자 정보
            *   API 호출 시간
            *   호출자의 IP 주소
            *   사용된 API 메서드
            *   요청/응답 세부 정보
        *   보안 감사 및 규정 준수를 위한 로그 보존
        *   API 호출 패턴 분석을 통한 보안 위협 탐지

    *   **X-Ray 통합**
        *   분산 추적을 통한 성능 분석
        *   서비스 간 지연 시간 측정
        *   오류 발생 지점 식별
        *   병목 현상 파악

    *   **CloudWatch Logs Insights 활용**
        *   로그 패턴 분석
        *   오류 추적 및 디버깅
        *   성능 분석

    *   **운영 모범 사례**
        *   정기적인 메트릭스 검토
        *   임계값 기반 자동 알림 설정
        *   로그 보존 기간 정책 수립
        *   보안 감사를 위한 CloudTrail 로그 분석
        *   성능 최적화를 위한 X-Ray 트레이스 분석

*   **보안 및 암호화**
    *   **암호화**
        *   저장 데이터 암호화: AWS KMS 사용
        *   전송 중 데이터 암호화: SSL/TLS
    *   **인증 및 권한 부여**
        *   IAM 정책을 통한 세밀한 액세스 제어
        *   VPC 엔드포인트 지원
        *   IAM 조건을 사용한 속성 기반 접근 제어
    *   **세분화된 액세스 제어**
        *   **항목 수준 액세스 제어**
            *   `dynamodb:LeadingKeys` 조건 키를 사용하여 파티션 키 기반의 항목 접근 제어
            *   예: 게임 앱에서 사용자가 자신의 게임 데이터만 접근 가능하도록 제한
            *   ID 공급자 역할과 연결된 IAM 정책에 조건 키 추가
        *   **속성 수준 액세스 제어**
            *   `dynamodb:Attributes` 조건 키로 특정 속성에 대한 접근 제어
            *   예: 항공편 정보 앱에서 조종사 이름, 승객 수 등 민감한 속성 숨김
        *   **웹 ID 페더레이션 지원**
            *   Amazon, Facebook, Google 등의 ID 공급자를 통한 인증 지원
            *   인증된 사용자의 세분화된 액세스 제어 가능
    *   **모니터링**
        *   CloudWatch를 통한 메트릭 모니터링
        *   CloudTrail을 통한 API 호출 로깅

*   **백업 및 복구**
    *   **온디맨드 백업**
        *   전체 테이블 백업
        *   성능이나 가용성에 영향 없음
        *   원하는 시점으로 복구 가능
    *   **특정 시점 복구 (PITR)**
        *   최근 35일 내 원하는 시점으로 복구
        *   연속 백업
        *   우발적인 쓰기/삭제 작업 복구에 유용

*   **모범 사례 및 성능 최적화**
    *   **파티션 키 설계**
        *   높은 카디널리티(고유성)를 가진 속성 선택
        *   핫 파티션 방지를 위한 쓰기 샤딩 구현
    *   **정렬 키 활용**
        *   계층적 데이터 구조 구현
        *   범위 쿼리 최적화
    *   **비용 최적화**
        *   적절한 용량 모드 선택
        *   Auto Scaling 활용
        *   예약 용량 구매 고려
    *   **애플리케이션 설계**
        *   지수 백오프를 통한 재시도 구현
        *   배치 작업 활용
        *   DAX를 통한 읽기 성능 향상

    *   **DynamoDB Accelerator (DAX)**
        *   **기본 특징**
            *   완전 관리형 인메모리 캐싱 서비스
            *   마이크로초 단위의 응답 시간 제공
            *   읽기 집약적 워크로드에 최적화
            *   기존 DynamoDB API와 완벽한 호환성
        *   **주요 기능**
            *   쓰기 캐시(write-through cache) 지원
            *   자동 데이터 동기화 및 캐시 무효화
            *   저장 데이터 및 전송 데이터 암호화
            *   초당 수백만 건의 요청 처리 가능
        *   **사용 사례**
            *   실시간 입찰 시스템
            *   소셜 게임
            *   거래 플랫폼
            *   빠른 데이터 액세스가 필요한 애플리케이션
        *   **장점**
            *   운영 복잡성 감소
            *   최소한의 코드 변경으로 구현 가능
            *   자동화된 클러스터 유지 관리
            *   데이터 일관성 보장

*   **오류 처리 및 재시도 전략**
    *   **일반적인 DynamoDB 오류**
        *   `ProvisionedThroughputExceededException`: 프로비저닝된 처리량 초과
        *   `ThrottlingException`: 요청이 너무 많음
        *   `ResourceNotFoundException`: 요청된 리소스가 없음
        *   `ConditionalCheckFailedException`: 조건부 쓰기 실패
    
    *   **오류 재시도와 지수 백오프**
        *   **기본 개념**
            *   재시도(Retry): 실패한 요청을 다시 시도
            *   지수 백오프(Exponential Backoff): 재시도 간격을 점진적으로 증가
            *   Jitter: 무작위 지연을 추가하여 경쟁 상태 방지
        
        *   **구현 예시**
            ```python
            import time
            import random
            import boto3
            from botocore.exceptions import ClientError

            def get_item_with_backoff(table_name, key, max_retries=5):
                dynamodb = boto3.resource('dynamodb')
                table = dynamodb.Table(table_name)
                retries = 0
                while True:
                    try:
                        return table.get_item(Key=key)
                    except ClientError as e:
                        if e.response['Error']['Code'] != 'ProvisionedThroughputExceededException':
                            raise
                        if retries >= max_retries:
                            raise
                        sleep_time = (2 ** retries * 100) + random.randint(0, 100)
                        time.sleep(sleep_time / 1000.0)
                        retries += 1
            ```

        *   **재시도 전략**
            *   첫 번째 재시도: 100ms 대기
            *   두 번째 재시도: 200ms 대기
            *   세 번째 재시도: 400ms 대기
            *   네 번째 재시도: 800ms 대기
            *   다섯 번째 재시도: 1600ms 대기
            *   각 재시도마다 무작위 지연(0-100ms) 추가

        *   **AWS SDK의 자동 재시도**
            *   대부분의 AWS SDK는 기본적으로 재시도 로직 내장
            *   SDK별 재시도 설정 방법:
                ```java
                // Java SDK 예시
                ClientConfiguration config = new ClientConfiguration()
                    .withMaxErrorRetry(10)
                    .withRetryPolicy(PredefinedRetryPolicies.getDynamoDBDefaultRetryPolicy());
                ```
                ```python
                # Python SDK (boto3) 예시
                config = Config(
                    retries = dict(
                        max_attempts = 10,
                        mode = 'adaptive'
                    )
                )
                ```

        *   **모범 사례**
            *   적절한 최대 재시도 횟수 설정
            *   충분한 타임아웃 값 설정
            *   오류 로깅 및 모니터링 구현
            *   배치 작업에서의 부분 실패 처리
            *   클라이언트 측 타임아웃 설정

    *   **성능 최적화 전략**
        *   배치 작업 사용으로 네트워크 왕복 최소화
        *   파티션 키 분산으로 핫 파티션 방지
        *   적절한 프로비저닝 용량 설정
        *   DAX 사용 고려 (읽기 중심 워크로드)
        *   글로벌 테이블 사용 (지역 분산 필요 시)

*   **용량 관리 및 모니터링**
    *   **ReturnConsumedCapacity 파라미터 활용**
        *   `NONE`: 기본값, 용량 소비 정보를 반환하지 않음
        *   `TOTAL`: 전체 소비된 용량 단위만 반환
        *   `INDEXES`: 테이블과 보조 인덱스별 상세 소비 용량 정보 반환
        *   사용 예시:
            ```python
            # 용량 소비 추적을 위한 UpdateItem 예시
            response = table.update_item(
                Key={
                    'partition_key': 'value1',
                    'sort_key': 'value2'
                },
                UpdateExpression='SET #attr = :val',
                ExpressionAttributeNames={
                    '#attr': 'some_attribute'
                },
                ExpressionAttributeValues={
                    ':val': 'new_value'
                },
                ReturnConsumedCapacity='INDEXES'
            )
            
            # 용량 소비 분석
            if 'ConsumedCapacity' in response:
                capacity = response['ConsumedCapacity']
                print(f"총 소비 용량: {capacity.get('CapacityUnits', 0)} 유닛")
                
                # 테이블 용량 소비
                if 'Table' in capacity:
                    print(f"테이블 용량 소비: {capacity['Table'].get('CapacityUnits', 0)} 유닛")
                
                # GSI 용량 소비
                if 'GlobalSecondaryIndexes' in capacity:
                    for gsi_name, gsi_capacity in capacity['GlobalSecondaryIndexes'].items():
                        print(f"GSI {gsi_name} 용량 소비: {gsi_capacity.get('CapacityUnits', 0)} 유닛")
            ```

    *   **CloudWatch를 통한 용량 모니터링**
        *   주요 메트릭스:
            *   `ConsumedReadCapacityUnits`: 소비된 읽기 용량 단위
            *   `ConsumedWriteCapacityUnits`: 소비된 쓰기 용량 단위
            *   `ThrottledRequests`: 제한된 요청 수
        *   모니터링 예시:
            ```python
            import boto3
            from datetime import datetime, timedelta

            def monitor_capacity_metrics(table_name):
                cloudwatch = boto3.client('cloudwatch')
                end_time = datetime.utcnow()
                start_time = end_time - timedelta(hours=1)
                
                metrics = cloudwatch.get_metric_data(
                    MetricDataQueries=[
                        {
                            'Id': 'consumedWriteCapacity',
                            'MetricStat': {
                                'Metric': {
                                    'Namespace': 'AWS/DynamoDB',
                                    'MetricName': 'ConsumedWriteCapacityUnits',
                                    'Dimensions': [
                                        {
                                            'Name': 'TableName',
                                            'Value': table_name
                                        }
                                    ]
                                },
                                'Period': 300,  # 5분 간격
                                'Stat': 'Sum'
                            }
                        }
                    ],
                    StartTime=start_time,
                    EndTime=end_time
                )
                return metrics
            ```

    *   **적응형 재시도 전략**
        *   boto3 설정:
            ```python
            from botocore.config import Config

            config = Config(
                retries = dict(
                    max_attempts = 10,  # 최대 재시도 횟수
                    mode = 'adaptive'   # 적응형 재시도 모드
                )
            )

            # DynamoDB 클라이언트 생성
            dynamodb = boto3.resource('dynamodb', config=config)
            ```
        
        *   재시도 전략 특징:
            *   자동으로 재시도 간격 조정
            *   서비스 상태에 따른 지능적 대기 시간 설정
            *   네트워크 지연과 서비스 부하 고려
            *   지수 백오프와 지터 자동 적용

    *   **용량 최적화 모범 사례**
        *   적절한 파티션 키 선택으로 용량 분산
        *   Auto Scaling 설정을 통한 자동 용량 조정
        *   배치 작업을 통한 용량 효율성 향상
        *   피크 시간대 용량 계획 수립
        *   정기적인 용량 사용량 분석 및 조정

*   **핫 파티션 문제와 해결 방안**
    *   **핫 파티션이란?**
        *   특정 파티션에 트래픽이 집중되는 현상
        *   프로비저닝된 처리량의 비효율적 사용 초래
        *   전체 테이블 성능 저하의 주요 원인
        *   스로틀링(throttling) 발생 가능성 증가

    *   **핫 파티션 발생 원인**
        *   부적절한 파티션 키 선택
            *   낮은 카디널리티(고유성)를 가진 키 사용
            *   특정 값에 편중된 접근 패턴
            *   순차적인 값을 파티션 키로 사용
        *   불균형한 데이터 접근
            *   특정 시간대의 집중적인 접근
            *   인기 있는 아이템에 대한 과도한 요청
        *   부적절한 용량 설정
            *   파티션별 처리량 한계 미고려
            *   Auto Scaling 설정 부재

    *   **파티션 키 선택 기준**
        *   높은 카디널리티(고유성)
            *   UUID, GUID 사용 권장
            *   타임스탬프 + 랜덤값 조합
            *   해시된 비즈니스 키
        *   균등한 데이터 분산
            *   순차적 값 회피
            *   고른 접근 패턴 보장
        *   비즈니스 요구사항 충족
            *   쿼리 패턴 고려
            *   확장성 고려

    *   **해결 방안 예시**
        ```python
        import uuid
        from datetime import datetime

        # 좋은 파티션 키 생성 예시
        def generate_partition_key():
            # UUID를 사용한 고유 식별자 생성
            return str(uuid.uuid4())

        # 타임스탬프와 랜덤값을 조합한 파티션 키
        def generate_timestamp_partition_key():
            timestamp = datetime.utcnow().strftime('%Y%m%d%H%M%S')
            random_suffix = str(uuid.uuid4())[:8]
            return f"{timestamp}#{random_suffix}"

        # 해시 기반 파티션 키 (비즈니스 키 활용)
        def generate_hashed_partition_key(business_key):
            import hashlib
            return hashlib.md5(business_key.encode()).hexdigest()
        ```

    *   **모니터링 및 예방**
        *   CloudWatch 메트릭스 활용
            *   `ConsumedReadCapacityUnits`
            *   `ConsumedWriteCapacityUnits`
            *   `ThrottledRequests`
        *   파티션 사용량 분석
            ```python
            def analyze_partition_usage(table_name):
                cloudwatch = boto3.client('cloudwatch')
                
                # 파티션별 사용량 메트릭스 수집
                response = cloudwatch.get_metric_statistics(
                    Namespace='AWS/DynamoDB',
                    MetricName='ConsumedWriteCapacityUnits',
                    Dimensions=[
                        {
                            'Name': 'TableName',
                            'Value': table_name
                        }
                    ],
                    StartTime=datetime.utcnow() - timedelta(hours=24),
                    EndTime=datetime.utcnow(),
                    Period=300,  # 5분 간격
                    Statistics=['Sum']
                )
                
                return response['Datapoints']
            ```

    *   **핫 파티션 방지를 위한 모범 사례**
        *   쓰기 샤딩 구현
            *   랜덤 접두사/접미사 사용
            *   해시 기반 분산
        *   적절한 파티션 키 조합 사용
            *   복합 키 활용
            *   GSI 활용
        *   캐싱 전략 도입
            *   DAX 활용
            *   인메모리 캐시 사용
        *   Auto Scaling 설정
            *   적절한 최소/최대 용량 설정
            *   대상 사용률 조정

    *   **실제 사례 시나리오**
        *   **온라인 교육 포털 예시**
            ```python
            # 좋은 설계 예시
            class CourseItem:
                def __init__(self, course_name, provider_id, price):
                    self.item_id = str(uuid.uuid4())  # 고유한 항목 ID
                    self.course_name = course_name
                    self.provider_id = provider_id
                    self.price = price
                    self.created_at = datetime.utcnow().isoformat()

            # DynamoDB 항목 생성
            def create_course_item(course_data):
                table.put_item(
                    Item={
                        'PK': f"COURSE#{generate_partition_key()}",  # 고유한 파티션 키
                        'SK': f"PROVIDER#{course_data['provider_id']}",  # 정렬 키
                        'course_name': course_data['course_name'],
                        'price': course_data['price'],
                        'created_at': datetime.utcnow().isoformat()
                    }
                )
            ```

        *   **접근 패턴 최적화**
            ```python
            # GSI를 활용한 다양한 접근 패턴 지원
            def create_course_with_gsi(course_data):
                item_id = generate_partition_key()
                table.put_item(
                    Item={
                        'PK': f"COURSE#{item_id}",
                        'SK': f"PROVIDER#{course_data['provider_id']}",
                        'GSI1PK': f"PROVIDER#{course_data['provider_id']}",
                        'GSI1SK': course_data['course_name'],
                        'GSI2PK': f"PRICE#{str(course_data['price']).zfill(10)}",
                        'GSI2SK': item_id,
                        # ... 기타 속성들
                    }
                )
            ```

*   **DynamoDB 샤딩 전략**
    *   **샤딩이란?**
        *   데이터를 여러 파티션에 분산하여 저장하는 기술
        *   단일 파티션의 부하를 줄이고 전체 성능을 향상
        *   데이터 접근을 고르게 분산하여 핫 파티션 방지
        *   확장성과 가용성 향상을 위한 핵심 전략

    *   **샤딩 패턴**
        *   **랜덤 접두사/접미사**
            ```python
            import random

            def create_item_with_random_shard(item_data):
                # 1-10 사이의 랜덤 샤드 번호 생성
                shard_id = random.randint(1, 10)
                
                table.put_item(
                    Item={
                        'PK': f"SHARD_{shard_id}#ITEM#{item_data['id']}",
                        'SK': item_data['sort_key'],
                        # ... 기타 속성들
                    }
                )
            ```

        *   **계산된 샤드 번호**
            ```python
            def create_item_with_calculated_shard(item_data):
                # 항목 ID를 기반으로 일관된 샤드 번호 계산
                shard_id = hash(item_data['id']) % 10 + 1
                
                return f"SHARD_{shard_id}#ITEM#{item_data['id']}"
            ```

        *   **타임스탬프 기반 샤딩**
            ```python
            from datetime import datetime

            def create_item_with_time_shard(item_data):
                # 현재 시간을 기준으로 샤드 키 생성
                current_hour = datetime.utcnow().strftime('%Y%m%d%H')
                
                table.put_item(
                    Item={
                        'PK': f"TIME_{current_hour}#{item_data['id']}",
                        'SK': item_data['sort_key'],
                        # ... 기타 속성들
                    }
                )
            ```

    *   **샤딩 구현 예시**
        *   **Write Sharding (쓰기 샤딩)**
            ```python
            class WriteShardingExample:
                def __init__(self, table_name, shard_count=10):
                    self.table = boto3.resource('dynamodb').Table(table_name)
                    self.shard_count = shard_count

                def write_item(self, user_id, data):
                    # 사용자 ID를 기반으로 샤드 결정
                    shard_id = hash(user_id) % self.shard_count
                    
                    self.table.put_item(
                        Item={
                            'PK': f"SHARD_{shard_id}#USER#{user_id}",
                            'SK': data['timestamp'],
                            'data': data['content']
                        }
                    )

                def read_item(self, user_id, timestamp):
                    # 동일한 해시 함수로 샤드 결정
                    shard_id = hash(user_id) % self.shard_count
                    
                    return self.table.get_item(
                        Key={
                            'PK': f"SHARD_{shard_id}#USER#{user_id}",
                            'SK': timestamp
                        }
                    )
            ```

        *   **Read Sharding (읽기 샤딩)**
            ```python
            class ReadShardingExample:
                def __init__(self, table_name, shard_count=10):
                    self.table = boto3.resource('dynamodb').Table(table_name)
                    self.shard_count = shard_count

                def write_to_all_shards(self, item_id, data):
                    # 모든 샤드에 데이터 복제
                    for shard_id in range(self.shard_count):
                        self.table.put_item(
                            Item={
                                'PK': f"SHARD_{shard_id}#ITEM#{item_id}",
                                'data': data
                            }
                        )

                def read_from_random_shard(self, item_id):
                    # 랜덤 샤드에서 읽기
                    shard_id = random.randint(0, self.shard_count - 1)
                    return self.table.get_item(
                        Key={
                            'PK': f"SHARD_{shard_id}#ITEM#{item_id}"
                        }
                    )
            ```

    *   **샤딩 사용 시 고려사항**
        *   **장점**
            *   트래픽 분산
            *   핫 파티션 방지
            *   처리량 향상
            *   비용 효율성 증가
        
        *   **주의사항**
            *   데이터 일관성 관리 필요
            *   복잡성 증가
            *   쿼리 패턴 영향
            *   샤드 수 결정의 중요성

    *   **샤딩 모범 사례**
        *   적절한 샤드 수 선택
        *   일관된 해시 함수 사용
        *   샤드 키 설계 시 확장성 고려
        *   모니터링 및 재샤딩 전략 수립
        *   백업 및 복구 계획 수립

*   **낙관적 잠금(Optimistic Locking) 구현**
    *   **개요**
        *   동시성 제어를 위한 버전 기반 잠금 전략
        *   데이터 일관성 보장 및 동시 수정 충돌 방지
        *   글로벌 테이블과 함께 사용 시 주의 필요 (최종 작성자 우선 정책과 충돌)
    
    *   **구현 방법**
        *   테이블 매핑 클래스에 버전 속성 추가
        *   조건부 업데이트를 통한 버전 검증
        *   충돌 발생 시 재시도 로직 구현
        ```python
        def update_with_optimistic_lock(table, item_id, new_data, expected_version):
            try:
                response = table.update_item(
                    Key={'id': item_id},
                    UpdateExpression='SET #data = :new_data, #version = :new_version',
                    ConditionExpression='#version = :expected_version',
                    ExpressionAttributeNames={
                        '#data': 'data',
                        '#version': 'version'
                    },
                    ExpressionAttributeValues={
                        ':new_data': new_data,
                        ':expected_version': expected_version,
                        ':new_version': expected_version + 1
                    }
                )
                return True
            except ClientError as e:
                if e.response['Error']['Code'] == 'ConditionalCheckFailedException':
                    return False  # 버전 불일치로 업데이트 실패
                raise e
        ```

    *   **재시도 메커니즘**
        *   지수 백오프 적용
        *   최대 재시도 횟수 제한
        *   사용자 친화적 에러 처리
        ```python
        def update_with_retry(table, item_id, new_data, max_retries=3):
            retries = 0
            while retries < max_retries:
                # 현재 항목 조회
                current_item = table.get_item(Key={'id': item_id})['Item']
                current_version = current_item['version']
                
                # 업데이트 시도
                if update_with_optimistic_lock(table, item_id, new_data, current_version):
                    return True
                
                retries += 1
                time.sleep((2 ** retries) * 0.1)  # 지수 백오프
            
            raise Exception("최대 재시도 횟수 초과")
        ```

    *   **장점**
        *   데이터 일관성 보장
        *   동시 수정 충돌 방지
        *   리소스 효율적 사용
        *   구현 및 유지보수 용이

    *   **주의사항**
        *   글로벌 테이블 사용 시 제한사항 존재
        *   DynamoDBMapper 트랜잭션 작업과 호환성 없음
        *   높은 동시성 환경에서 재시도 로직 튜닝 필요
        *   버전 관리로 인한 저장 공간 오버헤드