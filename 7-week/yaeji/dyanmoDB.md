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
        *   `BatchGetItem`: 최대 100개 항목 조회
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

*   **보안 및 암호화**
    *   **암호화**
        *   저장 데이터 암호화: AWS KMS 사용
        *   전송 중 데이터 암호화: SSL/TLS
    *   **인증 및 권한 부여**
        *   IAM 정책을 통한 세밀한 액세스 제어
        *   VPC 엔드포인트 지원
        *   IAM 조건을 사용한 속성 기반 접근 제어
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