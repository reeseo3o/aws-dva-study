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
        *   각 파티션 키 값당 10GB 제한
        *   기본 테이블의 용량을 공유
    *   **글로벌 보조 인덱스 (GSI)**
        *   언제든지 생성/삭제 가능
        *   다른 파티션 키와 정렬 키 사용 가능
        *   최종적 일관된 읽기만 지원
        *   테이블당 최대 20개 생성 가능
        *   별도의 RCU/WCU 설정 필요
        *   크기 제한 없음
        *   비동기 업데이트로 인한 데이터 일관성 지연 가능

    *   **인덱스 프로젝션 유형**
        *   `KEYS_ONLY`: 
            *   인덱스 키와 기본 테이블의 키만 포함
            *   최소 스토리지 비용
            *   키 기반 조회만 필요한 경우 적합
        *   `INCLUDE`: 
            *   지정된 속성만 선택적으로 포함
            *   스토리지와 성능의 균형 필요 시 사용
            *   특정 속성만 자주 조회하는 경우 적합
        *   `ALL`: 
            *   모든 속성을 포함
            *   추가 GetItem 호출 회피 필요 시 사용
            *   모든 속성에 대한 빈번한 조회 시 적합

    *   **인덱스 선택 가이드라인**
        *   **LSI 선택 기준**:
            *   강력한 일관성이 필수적인 경우
            *   단일 파티션 내 쿼리가 대부분인 경우
            *   비용 최적화가 중요한 경우
            *   파티션 키당 데이터가 10GB 미만인 경우
        *   **GSI 선택 기준**:
            *   테이블 전체 범위의 쿼리가 필요한 경우
            *   유연한 키 구성이 필요한 경우
            *   최종적 일관성으로 충분한 경우
            *   데이터 크기가 큰 경우

    *   **인덱스 성능 최적화**
        *   적절한 프로젝션 유형 선택
        *   쿼리 패턴에 맞는 키 구성
        *   배치 작업 활용
        *   페이지네이션 구현
        *   필요한 속성만 프로젝션

    *   **인덱스 비용 고려사항**
        *   **LSI 비용 요소**:
            *   기본 테이블의 WCU/RCU 공유
            *   추가 스토리지 비용 (프로젝션된 속성에 따라)
            *   파티션 키당 10GB 제한으로 인한 설계 제약
        *   **GSI 비용 요소**:
            *   별도의 WCU/RCU 비용
            *   프로젝션된 속성에 따른 스토리지 비용
            *   데이터 복제로 인한 추가 비용

    *   **인덱스 모니터링**
        *   CloudWatch 메트릭스 활용
            *   ConsumedReadCapacityUnits
            *   ConsumedWriteCapacityUnits
            *   ThrottledRequests
        *   성능 지표 모니터링
        *   용량 사용량 추적
        *   스로틀링 이벤트 감지

*   **DynamoDB Streams**
    *   테이블의 데이터 수정 이벤트를 시간 순서대로 기록
    *   최대 24시간 동안 데이터 보존
    *   네 가지 보기 유형:
        *   KEYS_ONLY: 수정된 항목의 키만 기록
        *   NEW_IMAGE: 수정 후의 전체 항목 이미지
        *   OLD_IMAGE: 수정 전의 전체 항목 이미지
        *   NEW_AND_OLD_IMAGES: 수정 전후의 항목 이미지
    *   Lambda 함수와 통합하여 이벤트 기반 아키텍처 구현
        *   **Lambda 함수 호출 유형**
            *   동기(Synchronous) 호출
                *   `RequestResponse` (기본값)
                *   함수 실행 완료까지 대기
                *   응답에 함수 실행 결과 포함
                *   실시간 처리가 필요한 경우 적합
            *   비동기(Asynchronous) 호출
                *   `Event` 타입으로 호출
                *   즉시 응답 반환 (202 상태 코드)
                *   백그라운드에서 함수 실행
                *   자동 재시도 메커니즘 제공
                *   Dead Letter Queue 구성 가능
                *   대량 처리나 시간이 오래 걸리는 작업에 적합
            *   구현 예시:
                ```python
                import boto3

                # Lambda 클라이언트 생성
                lambda_client = boto3.client('lambda')

                # 비동기 호출
                async_response = lambda_client.invoke(
                    FunctionName='YourFunctionName',
                    InvocationType='Event',
                    Payload='{"key": "value"}'
                )

                # 동기 호출
                sync_response = lambda_client.invoke(
                    FunctionName='YourFunctionName',
                    InvocationType='RequestResponse',
                    Payload='{"key": "value"}'
                )
                ```
            *   주의사항
                *   InvokeAsync API는 더 이상 사용되지 않음
                *   비동기 호출은 Invoke API + Event 타입 사용
                *   오류 처리를 위한 DLQ 구성 권장
                *   재시도 정책 설정 필요
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

*   **DynamoDB 캐싱 전략**
    *   **캐싱 전략 유형**
        *   **Lazy Loading (지연 로딩)**
            *   특징:
                *   캐시 미스가 발생할 때만 데이터를 캐시에 로드
                *   필요한 데이터만 캐시에 저장
                *   캐시의 데이터가 최신 상태가 아닐 수 있음
            *   장점:
                *   불필요한 데이터가 캐시되지 않음
                *   캐시 미스 시에만 쓰기 발생
            *   단점:
                *   캐시 미스 시 지연 시간 증가
                *   데이터가 오래된 상태일 수 있음
        
        *   **Write Through (연속 쓰기)**
            *   특징:
                *   데이터베이스에 쓸 때마다 캐시도 업데이트
                *   캐시의 데이터가 항상 최신 상태 유지
                *   데이터 일관성 보장
            *   장점:
                *   캐시가 항상 최신 상태
                *   읽기 지연 시간 감소
            *   단점:
                *   쓰기 지연 시간 증가
                *   사용되지 않는 데이터도 캐시에 저장
                *   리소스 및 공간 낭비 가능성

        *   **Write Through + TTL**
            *   특징:
                *   Write Through 전략에 TTL(Time To Live) 추가
                *   일정 시간 후 캐시 데이터 자동 삭제
                *   공간 효율성과 데이터 최신성 모두 확보
            *   장점:
                *   캐시 데이터 최신성 보장
                *   미사용 데이터 자동 제거
                *   리소스 효율적 사용
            *   구현 예시:
                ```python
                # ElastiCache Redis 설정 예시
                redis_client = redis.Redis(
                    host='your-elasticache-endpoint',
                    port=6379,
                    decode_responses=True
                )

                def write_through_with_ttl(key, value, ttl_seconds=3600):
                    # DynamoDB에 데이터 쓰기
                    dynamodb_table.put_item(
                        Item={
                            'id': key,
                            'data': value
                        }
                    )
                    
                    # 캐시에 데이터 쓰기 (TTL 설정)
                    redis_client.setex(
                        name=key,
                        time=ttl_seconds,
                        value=json.dumps(value)
                    )

                def read_data(key):
                    # 캐시에서 먼저 조회
                    cached_data = redis_client.get(key)
                    if cached_data:
                        return json.loads(cached_data)
                    
                    # 캐시 미스 시 DynamoDB에서 조회
                    response = dynamodb_table.get_item(
                        Key={'id': key}
                    )
                    if 'Item' in response:
                        return response['Item']['data']
                    return None
                ```

    *   **캐싱 전략 선택 기준**
        *   **Lazy Loading 적합한 경우**:
            *   데이터가 자주 변경되지 않는 경우
            *   일부 오래된 데이터가 허용되는 경우
            *   캐시 미스로 인한 지연이 허용되는 경우
        
        *   **Write Through 적합한 경우**:
            *   데이터 일관성이 중요한 경우
            *   읽기가 빈번한 경우
            *   쓰기 지연이 허용되는 경우
        
        *   **Write Through + TTL 적합한 경우**:
            *   데이터 최신성이 중요한 경우
            *   캐시 공간 효율성이 중요한 경우
            *   주기적인 데이터 갱신이 필요한 경우

    *   **캐싱 모범 사례**
        *   적절한 TTL 값 설정
        *   캐시 크기 모니터링
        *   장애 복구 전략 수립
        *   캐시 무효화 정책 수립
        *   성능 메트릭 모니터링

    *   **주의사항**
        *   캐시 데이터 정합성 관리
        *   메모리 사용량 모니터링
        *   네트워크 지연 고려
        *   장애 상황 대비
        *   비용 효율성 분석