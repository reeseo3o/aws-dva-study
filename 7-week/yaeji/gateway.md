## 1. API Gateway 소개

### 1.1. API Gateway란?
- AWS에서 제공하는 완전관리형 서비스로, 어떤 규모에서든 API를 손쉽게 생성, 게시, 유지 관리, 모니터링 및 보호할 수 있게 해줍니다.
- RESTful API 및 WebSocket API를 생성하여 백엔드 서비스(AWS Lambda, Amazon EC2, 온프레미스 애플리케이션 등)와 통합할 수 있습니다.

### 1.2. 주요 기능 및 이점
- **서버리스**: 인프라 관리 없이 API 운영 가능.
- **통합**: Lambda, HTTP 엔드포인트, 기타 AWS 서비스와 쉽게 통합.
- **보안**: IAM, Amazon Cognito, Lambda Authorizer를 통한 인증 및 권한 부여.
- **트래픽 관리**: 스로틀링, 할당량(쿼터) 설정, API 키 관리.
- **버전 관리 및 스테이지**: API의 여러 버전을 여러 환경(dev, test, prod)에서 관리.
- **캐싱**: API 응답 캐싱으로 지연 시간 감소 및 백엔드 부하 절감.
- **모니터링**: CloudWatch를 통한 API 호출, 지연 시간, 오류율 등 모니터링.
- **SDK 생성**: 다양한 프로그래밍 언어로 SDK 자동 생성.
- **OpenAPI (Swagger) 지원**: API 정의 가져오기 및 내보내기.
- **요청/응답 변환**: 매핑 템플릿을 사용하여 요청과 응답 데이터 형식 변환.

## 2. API Gateway 핵심 구성 요소

### 2.1. 엔드포인트 유형
API 게이트웨이를 배포하는 방식으로, 클라이언트가 API에 액세스하는 방법을 결정합니다.
- **Edge-Optimized (엣지 최적화)**:
    - 글로벌 클라이언트에 적합.
    - Amazon CloudFront 엣지 로케이션을 통해 요청을 라우팅하여 지연 시간 개선.
    - API 게이트웨이는 특정 리전에 존재하지만, 전 세계 CloudFront 엣지에서 액세스 가능.
- **Regional (리전)**:
    - API 게이트웨이가 생성된 리전 내에 있는 클라이언트에 적합.
    - 자체 CloudFront 배포를 생성하여 엣지 최적화와 유사한 효과 및 추가 제어 가능.
- **Private (프라이빗)**:
    - VPC(Virtual Private Cloud) 내에서만 액세스 가능.
    - 인터페이스 VPC 엔드포인트(ENI)를 사용.
    - 리소스 정책을 통해 액세스 제어.

### 2.2. 통합 유형
API 게이트웨이가 백엔드와 통신하는 방식입니다.
- **Lambda 함수 통합**:
    - 가장 일반적인 서버리스 애플리케이션 패턴.
    - **Lambda 프록시 통합 (Lambda Proxy Integration)**:
        - API Gateway가 클라이언트의 전체 HTTP 요청을 Lambda 함수에 그대로 전달.
        - Lambda 함수는 특정 형식의 JSON 응답을 반환해야 함.
        - 매핑 템플릿이나 변환 없이 자동으로 처리됨.
        - **장점**:
            - 설정이 매우 간단함.
            - API Gateway 설정 변경 없이 Lambda 함수에서 모든 로직을 처리할 수 있음.
        - **단점**:
            - 요청/응답 형식을 API Gateway 수준에서 변경할 수 없음.
            - Lambda 함수가 API Gateway의 응답 형식을 정확히 따라y야 함.
    - **Lambda 사용자 정의 통합 (Lambda Custom Integration)**:
        - API Gateway에서 매핑 템플릿을 사용하여 요청과 응답을 변환할 수 있음.
        - 입력과 출력 형식을 완벽하게 제어할 수 있음.
        - VTL(Velocity Template Language)을 사용하여 매핑 템플릿을 작성.
        - **장점**:
            - 요청/응답 형식을 완벽하게 제어할 수 있음.
            - Lambda 함수는 API Gateway의 형식에 구애받지 않고 자유롭게 구현할 수 있음.
        - **단점**:
            - 설정이 복잡하고 VTL 문법을 이해해야 함.
            - 매핑 템플릿 관리에 추가적인 노력이 필요함.
    - **통합 방식 선택 가이드**:
        - Lambda 프록시 통합 선택 시기:
            - 빠른 API 프로토타이핑이 필요할 때
            - 단순한 REST API를 구현할 때
            - Lambda 함수에서 모든 로직을 처리하고 싶을 때
        - Lambda 사용자 정의 통합 선택 시기:
            - 요청/응답 형식의 세밀한 제어가 필요할 때
            - 기존 백엔드 시스템과의 호환성이 중요할 때
            - API Gateway 수준에서 요청/응답 변환이 필요할 때
- **HTTP 통합**:
    - 온프레미스 또는 클라우드의 HTTP 엔드포인트(예: Application Load Balancer)를 백엔드로 사용.
    - `HTTP_PROXY` (HTTP 프록시 통합): 요청을 백엔드로 직접 전달.
    - HTTP 직접 통합: 매핑 템플릿 사용 가능.
- **AWS 서비스 통합**:
    - API 게이트웨이를 통해 다른 AWS 서비스(예: Amazon SQS, Step Functions, Kinesis Data Streams) 직접 호출.
    - 인증 추가, 공개적 배포, 속도 제한 등의 목적으로 사용.
- **MOCK 통합**:
    - 실제 백엔드 호출 없이 API 게이트웨이가 응답 생성.
    - API 개발 및 테스트, CORS 사전 요청(preflight) 처리 등에 유용.

## 3. API 배포 및 관리

### 3.1. 스테이지 (Stages)
- API의 특정 배포 시점을 나타내며, 서로 다른 환경(예: `dev`, `test`, `prod`) 또는 버전(`v1`, `v2`)을 관리하는 데 사용됩니다.
- 각 스테이지는 고유한 구성(엔드포인트, 캐싱 설정, 스테이지 변수 등)을 가질 수 있습니다.
- API 변경 사항은 스테이지에 **배포**되어야 실제 적용됩니다.
- 롤백 기능: 이전 배포 버전으로 쉽게 되돌릴 수 있습니다.

### 3.2. 스테이지 변수 (Stage Variables)
- 스테이지별로 정의할 수 있는 환경 변수와 유사한 키-값 쌍.
- API를 재배포하지 않고도 구성 값을 변경할 수 있습니다 (예: Lambda 함수 별칭, HTTP 엔드포인트 URL).
- API 게이트웨이 구성에서는 `${stageVariables.variableName}` 형식으로, Lambda 함수 내에서는 컨텍스트 객체를 통해 액세스 가능.
- **주요 사용 사례**: 스테이지별로 다른 Lambda 함수 별칭(alias)을 가리키도록 설정.
    - 예: `dev` 스테이지 -> Lambda `DEV` 별칭, `prod` 스테이지 -> Lambda `PROD` 별칭.
    - Lambda 별칭을 업데이트하여 API 게이트웨이 재배포 없이 백엔드 Lambda 버전 변경 가능.

### 3.3. 카나리 릴리스 배포 (Canary Release Deployments)
- 프로덕션 스테이지에서 API의 새 버전을 소수의 트래픽에만 노출하여 테스트하는 방법.
- 트래픽 비율(예: 90%는 기존 버전, 10%는 카나리 버전)을 설정.
- 카나리 버전은 자체 스테이지 변수, 로깅, 지표 등을 가질 수 있습니다.
- 새 버전이 안정적이라고 판단되면 100% 트래픽을 카나리 버전으로 전환(승격)할 수 있습니다.

## 4. 보안

### 4.1. 인증 및 권한 부여 메커니즘
- **IAM (Identity and Access Management)**:
    - AWS 계정 내의 사용자, 역할, 서비스(EC2, Lambda 등)에 대한 인증 및 권한 부여.
    - SigV4 서명을 사용하여 요청 인증.
    - 주로 내부 애플리케이션 또는 AWS 서비스 간 호출에 사용.
- **Amazon Cognito User Pools**:
    - 웹 및 모바일 애플리케이션 사용자를 위한 완전관리형 사용자 디렉터리.
    - 사용자 가입, 로그인, 토큰 기반 인증 처리.
    - API 게이트웨이가 Cognito에서 발급한 토큰을 검증.
- **Lambda Authorizers (이전 명칭: Custom Authorizers)**:
    - 사용자 정의 Lambda 함수를 사용하여 인증 로직 구현.
    - 토큰 기반(Bearer 토큰, JWT, OAuth 등) 또는 요청 파라미터 기반 인증 가능.
    - Lambda 함수는 인증 성공 시 IAM 정책을 반환하며, 이 정책은 캐싱될 수 있음.
    - 서드파티 인증 시스템과의 통합 등 유연한 인증 처리에 적합.
    - **주요 특징**:
        - **캐싱 기능**:
            - 응답은 기본적으로 300초(5분) 동안 캐싱됨
            - TTL(Time To Live)을 0으로 설정하여 캐싱 비활성화 가능
            - 동일한 토큰/요청에 대한 반복적인 Lambda 호출 감소
        - **인증 워크플로우**:
            1. 클라이언트가 API 요청
            2. API Gateway가 Lambda Authorizer 호출
            3. Lambda Authorizer가 정책 문서 반환
            4. API Gateway가 정책을 평가
            5. 허용/거부 결정에 따라 API 메서드 실행 또는 403 Forbidden 반환
        - **인증 유형**:
            - **토큰 기반 Authorizer (TOKEN)**:
                - Bearer 토큰 인증에 적합
                - JWT나 OAuth 토큰 검증에 주로 사용
                - 헤더에서 토큰을 추출하여 검증
            - **요청 파라미터 기반 Authorizer (REQUEST)**:
                - 헤더, 쿼리 스트링 파라미터, 컨텍스트 변수, 스테이지 변수 조합으로 인증
                - IP 기반 화이트리스팅, 여러 헤더 조합 인증 등에 활용
        - **오류 처리**:
            - Lambda Authorizer 오류/타임아웃 시 403 Forbidden 응답 반환
            - 커스텀 오류 메시지 설정 가능
            - 기본 타임아웃 10초, 최대 30초까지 설정 가능
        - **보안 고려사항**:
            - Lambda Authorizer는 API Gateway에서 실행 권한 필요
            - 토큰/자격증명은 항상 HTTPS를 통해 전송
            - 민감한 인증 정보는 AWS Secrets Manager나 Parameter Store에 저장 권장
        - **모니터링과 로깅**:
            - CloudWatch Logs를 통한 Lambda Authorizer 로깅
            - CloudWatch Metrics를 통한 성능 모니터링
            - X-Ray를 통한 상세한 트레이싱 가능
        - **비용 고려사항**:
            - Lambda Authorizer 호출당 비용 발생
            - 캐싱을 통한 비용 최적화 가능
            - 인증 실패한 요청에도 Lambda 비용 발생
- **리소스 정책 (Resource Policies)**:
    - API 게이트웨이 자체에 연결되는 JSON 정책.
    - 교차 계정 액세스 허용, 특정 IP 주소 범위 허용/차단, 특정 VPC 엔드포인트에서의 호출만 허용 등의 접근 제어.

### 4.2. HTTPS
- AWS Certificate Manager (ACM)와 통합하여 사용자 지정 도메인 이름에 SSL/TLS 인증서 적용.
    - Edge-Optimized 엔드포인트: `us-east-1` 리전의 인증서 필요.
    - Regional 엔드포인트: API 게이트웨이와 동일 리전의 인증서 필요.
- Route 53을 사용하여 사용자 지정 도메인을 API 게이트웨이 엔드포인트에 매핑 (CNAME 또는 A-Alias 레코드).

## 5. 요청 및 응답 처리

### 5.1. 매핑 템플릿 (Mapping Templates)
- 통합 요청(Integration Request) 및 통합 응답(Integration Response)에서 데이터 형식을 변환하는 데 사용. (프록시 통합 제외)
- Apache Velocity Template Language (VTL) 사용.
- **기능**:
    - 요청/응답 본문 수정 (예: JSON에서 XML로 변환, 필드 이름 변경).
    - 헤더 추가/수정.
    - 쿼리 문자열 파라미터 이름 변경.
- **주요 사용 사례**:
    - SOAP API와 통합 시 JSON <-> XML 변환.
    - 백엔드가 요구하는 형식으로 요청 데이터 조정.
    - 클라이언트가 원하는 형식으로 응답 데이터 조정.

### 5.2. 요청 검증 (Request Validation)
- OpenAPI 정의를 사용하여 들어오는 요청의 파라미터(URI, 쿼리 문자열), 헤더, 본문이 지정된 스키마를 준수하는지 검증.
- 유효하지 않은 요청은 백엔드에 도달하기 전에 400 오류로 차단.
- API 수준 또는 개별 메서드 수준에서 설정 가능.

## 6. OpenAPI (Swagger) 사양
- REST API를 YAML 또는 JSON 형식으로 정의하는 표준.
- **기능**:
    - **가져오기(Import)**: OpenAPI 사양 파일을 사용하여 API 게이트웨이에서 API 생성 또는 업데이트.
    - **내보내기(Export)**: 기존 API 게이트웨이 구성을 OpenAPI 사양 파일로 내보내기. (클라이언트 SDK 생성, API 문서화 등에 활용)
    - 요청 검증에 활용.

## 7. 캐싱 (Caching)
- API 응답을 캐싱하여 백엔드 호출 수를 줄이고 지연 시간을 개선.
- **설정**:
    - 스테이지 수준에서 활성화하며, 메서드별로 재정의 가능.
    - TTL(Time To Live): 기본 300초(5분), 0초(캐싱 안 함) ~ 3600초(1시간) 설정 가능.
    - 캐시 용량: 0.5GB ~ 237GB. (비용 발생)
    - 캐시 데이터 암호화 가능.
- **무효화**:
    - 콘솔에서 전체 캐시 즉시 무효화.
    - 클라이언트가 `Cache-Control: max-age=0` 헤더를 전송하여 특정 캐시 항목 무효화 (IAM 권한 필요).

## 8. 사용량 계획 및 API 키
- API 사용량을 제어하고 관리하기 위한 기능.
- **API 키 (API Keys)**:
    - 클라이언트를 식별하는 문자열 값.
    - 클라이언트는 요청 시 `x-api-key` 헤더에 API 키를 포함.
    - 메서드 설정에서 API 키 필요 여부 지정.
- **사용량 계획 (Usage Plans)**:
    - 하나 이상의 배포된 API 스테이지에 대해 스로틀링 한도 및 할당량(쿼터) 설정.
        - **스로틀링**: 특정 기간 동안의 요청 속도(rate) 및 버스트(burst) 용량 제한.
        - **할당량**: 특정 기간(예: 월) 동안의 총 요청 수 제한.
    - API 키를 사용량 계획에 연결하여 고객별로 다른 사용 정책 적용.

## 9. 로깅 및 모니터링

### 9.1. CloudWatch Logs
- API 게이트웨이를 통과하는 요청 및 응답에 대한 상세 로깅.
- 스테이지 수준에서 활성화하며, 로그 수준(ERROR, INFO, DEBUG) 선택 가능.
- 요청/응답 본문, 헤더 등 기록 (민감 정보 포함 가능성에 유의).

### 9.2. AWS X-Ray
- API 게이트웨이 및 백엔드 서비스(Lambda 등)를 통과하는 요청에 대한 엔드투엔드 추적 정보 제공.

### 9.3. CloudWatch Metrics
- API 성능 및 사용량에 대한 지표 제공.
- **주요 지표**:
    - `Count`: API 요청 수.
    - `Latency`: 클라이언트가 요청을 받고 응답을 반환할 때까지의 총 시간. (API 게이트웨이 오버헤드 포함)
    - `IntegrationLatency`: API 게이트웨이가 백엔드로 요청을 보내고 응답을 받는 데 걸리는 시간.
    - `CacheHitCount`: 캐시에서 응답을 성공적으로 반환한 횟수.
    - `CacheMissCount`: 캐시에 해당 응답이 없어 백엔드를 호출한 횟수.
    - `4xxError`: 클라이언트 측 오류 수 (예: 잘못된 요청, 인증 실패).
    - `5xxError`: 서버 측 오류 수 (예: 백엔드 오류, 통합 시간 초과).
- **API 게이트웨이 시간 초과**: 최대 29초. 이 시간을 초과하면 504 오류 발생.

## 10. CORS (Cross-Origin Resource Sharing)
- 웹 브라우저 보안 기능으로, 한 도메인에서 실행 중인 웹 애플리케이션이 다른 도메인의 리소스에 접근할 수 있도록 허용하는 메커니즘.
- 다른 도메인에서 API를 호출하려면 CORS 활성화 필요.
- 브라우저는 실제 요청 전에 `OPTIONS` 메서드를 사용한 사전 전달(preflight) 요청을 보냄.
- API 게이트웨이는 `Access-Control-Allow-Origin`, `Access-Control-Allow-Methods`, `Access-Control-Allow-Headers` 등의 CORS 헤더를 응답해야 함.
- API 게이트웨이 콘솔에서 쉽게 설정 가능.

## 11. API 유형

### 11.1. REST API
- 이 문서에서 주로 다룬 API 유형으로, 다양한 고급 기능(매핑 템플릿, 캐싱, API 키, 사용량 계획 등) 제공.

### 11.2. HTTP API
- REST API보다 저렴하고 지연 시간이 낮은 경량 API.
- Lambda 프록시, HTTP 프록시, 프라이빗 통합에 적합.
- 데이터 매핑, 사용량 계획, API 키 등 일부 고급 기능 미지원.
- OIDC 및 OAuth 2.0 기본 인증 지원.

### 11.3. WebSocket API
- 클라이언트와 서버 간의 양방향 실시간 통신 지원.
- 상태 유지(Stateful) 연결.
- **주요 사용 사례**: 채팅 애플리케이션, 실시간 협업 도구, 멀티플레이어 게임, 금융 거래 플랫폼.
- **작동 방식**:
    - 클라이언트가 `wss://` URL을 통해 WebSocket 연결 시작.
    - `@connect`, `@disconnect`, `@default` 및 사용자 정의 라우트 키를 통해 백엔드(주로 Lambda)와 통합.
    - `connectionId`를 통해 각 클라이언트 연결 식별.
    - 서버는 `@connections/{connectionId}` 콜백 URL을 통해 클라이언트에게 메시지 푸시 가능.
    - 라우트 선택 표현식을 사용하여 수신 메시지의 내용에 따라 다른 백엔드 라우트로 전달.

## 12. 마이크로서비스 아키텍처와 API Gateway
- API 게이트웨이는 다양한 백엔드 마이크로서비스(ECS, ELB 뒤의 EC2, Lambda, S3 등)에 대한 단일 진입점(Single Entry Point) 역할.
- 클라이언트에게 복잡한 백엔드 아키텍처를 숨기고 일관된 API 인터페이스 제공.
- Route 53을 이용한 사용자 지정 도메인 및 SSL 인증서 적용.
- 경로 기반 라우팅, 요청 변환 등을 통해 유연한 서비스 통합 지원.