# Amazon S3 보안 정리

## 서버 측 암호화 (Server-Side Encryption, SSE)

### 1. SSE-S3 (Amazon S3 Managed Keys)
- AWS가 키를 관리하며, `AES-256` 암호화 방식 사용
- 요청 시 헤더 설정 필요: `x-amz-server-side-encryption: AES256`
- 객체는 Amazon S3의 관리 키로 암호화되며, 키는 사용자에게 노출되지 않음
- 새로운 객체 및 버킷에 대해 기본적으로 활성화됨

### 2. SSE-KMS (AWS Key Management Service)
- AWS KMS를 통해 키를 직접 생성/관리 가능
- 보안 이점: KMS 키에 대한 액세스 제어 및 로깅 제공 (CloudTrail 연동 가능)
- 요청 시 필요한 헤더:
  ```http
  x-amz-server-side-encryption: aws:kms
  x-amz-server-side-encryption-aws-kms-key-id: <KMS Key ID>
  ```
- 주의사항: KMS API 호출 수 제한 존재 (리전별 초당 5,000~30,000 요청)

### 3. SSE-C (Customer-Provided Keys)
- 사용자가 직접 암호화 키를 생성 및 제공
- AWS는 해당 키를 저장하지 않음
- 모든 요청 시 HTTPS 필수, 모든 요청의 헤더에 키를 달아야 함
- 암호화/복호화 시 동일 키 필요

## 💻 클라이언트 측 암호화 (Client-Side Encryption)
- 클라이언트가 데이터를 직접 암호화 후 S3에 업로드
- 복호화 또한 클라이언트에서 수행
- Amazon S3는 암호화된 데이터를 단순히 저장만 함
- 암호화 키와 로직은 클라이언트가 직접 관리
- AWS SDK 또는 암호화 라이브러리 사용 가능

## 🔄 전송 중 암호화 (Encryption in Transit)
- HTTPS(SSL/TLS) 사용을 권장
- 두 가지 엔드포인트 존재:
  - HTTP (비암호화)
  - HTTPS (암호화)
- SSE-C를 사용하는 경우 반드시 HTTPS 필수

### 전송 중 암호화 강제 방법
- 버킷 정책 예시:
  ```json
  {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Sid": "DenyUnEncryptedTransport",
        "Effect": "Deny",
        "Principal": "*",
        "Action": "s3:*",
        "Resource": [
          "arn:aws:s3:::<bucket-name>",
          "arn:aws:s3:::<bucket-name>/*"
        ],
        "Condition": {
          "Bool": {
            "aws:SecureTransport": "false"
          }
        }
      }
    ]
  }
  ```

## 요약
- 암호화 방식과 키 관리 주체:
  - **SSE-S3**: AWS 관리, 간편하지만 제어 불가
  - **SSE-KMS**: 사용자(KMS) 관리, 세밀한 제어 가능, 로그 기록
  - **SSE-C**: 사용자 직접 관리, 키 노출 주의 필요, HTTPS 필수
  - **클라이언트 암호화**: 사용자 관리, 전 과정 직접 관리, 가장 유연

## 시험 포인트
- 어떤 암호화 방식을 어떤 상황에 사용할지 구분할 수 있어야 함
- 각 암호화 방식의 키 관리 주체 및 제약 사항 숙지 필요
- 기본적으로 모든 버킷은 기본 암호화인 SSE-S3를 사용
- 버킷 정책을 통해 올바른 암호화 헤더가 없는 S3 객체에 적용되는 API 호출을 거부하여 암호화를 강제로 적용 가능

## 🌍 CORS (Cross-Origin Resource Sharing)

### CORS란?
- 교차 오리진 리소스 공유(Cross-Origin Resource Sharing)의 약자
- 웹 브라우저 보안 메커니즘으로, 한 오리진에서 시작된 웹 애플리케이션이 다른 오리진의 리소스에 접근할 수 있도록 허용할지 결정

### 오리진의 구성
- 오리진(Origin)은 다음 세 요소로 구성:
  - 프로토콜 (예: https)
  - 도메인 (예: www.example.com)
  - 포트 (예: 443, 80 등)

### 동일 오리진 vs 교차 오리진
- **동일 오리진**: https://www.example.com
- **교차 오리진**: https://other.example.com (도메인이 다름)

### CORS 동작 흐름
1. 웹 브라우저가 https://www.example.com의 index.html을 로드
2. index.html 내에서 https://www.other.com/images/coffee.jpg를 요청
3. 브라우저는 사전 요청(OPTIONS)을 www.other.com에 보냄
4. www.other.com 서버가 적절한 CORS 헤더를 포함해 응답 (Access-Control-Allow-Origin)
5. 브라우저가 정식 요청을 보내고 이미지 로딩 성공

## 🪣 Amazon S3에서의 CORS 적용

### 예시 시나리오
- **my-bucket-html** 버킷: 정적 웹사이트용 HTML 파일을 호스팅
- **my-bucket-assets** 버킷: 이미지 등의 리소스를 저장
- 웹 브라우저가 my-bucket-html의 index.html을 로드한 뒤, 그 내부에서 my-bucket-assets의 이미지(images/coffee.jpg)를 요청하는 경우
- 이때 my-bucket-assets 버킷에 CORS 설정이 없거나 잘못되어 있으면 요청이 거부됨

### S3 CORS 예시 설정
```xml
<CORSConfiguration>
  <CORSRule>
    <AllowedOrigin>*</AllowedOrigin> <!-- 모든 오리진 허용 -->
    <AllowedMethod>GET</AllowedMethod>
    <AllowedMethod>HEAD</AllowedMethod>
    <AllowedHeader>*</AllowedHeader>
  </CORSRule>
</CORSConfiguration>
```
- *는 모든 오리진을 허용하지만, 민감한 리소스에 대해서는 특정 오리진만 허용하도록 제한하는 것이 보안상 더 좋습니다.

## OPTIONS 요청이란?
- OPTIONS는 HTTP 메서드 중 하나로, "이 URL에 대해 어떤 HTTP 메서드와 헤더가 허용되나요?"라는 질문을 서버에 보내는 요청
- 즉, **"이 요청을 해도 돼요?"**라고 미리 물어보는 용도
- 이는 데이터를 실제로 요청하지 않기 때문에 **'안전한 요청'**입니다.

## 🚦 Preflight 요청이란?
- CORS에서는 **"Preflight Request"**라는 사전 요청이 필요할 수 있음
- 브라우저가 교차 오리진 요청을 보낼 때, 보안상 이유로 먼저 서버에게 "이런 요청을 보낼 건데 괜찮아?" 하고 묻는 과정

### Preflight 요청의 트리거 조건
- 브라우저는 다음 중 하나라도 해당하면 **Preflight 요청 (OPTIONS)**을 먼저 보냅니다:
  - GET, HEAD, POST 이외의 메서드를 사용할 때 (예: PUT, DELETE)
  - Content-Type이 application/json 등 "Simple Content-Type"이 아닐 때
  - 커스텀 헤더가 포함될 때 (예: X-Requested-With)

### 📦 Preflight 요청 구성
```http
OPTIONS /images/coffee.jpg HTTP/1.1
Origin: https://www.example.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: X-Custom-Header
```
- 이 요청에 대해 서버는 다음과 같이 응답해야 합니다:
```http
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://www.example.com
Access-Control-Allow-Methods: GET, PUT, POST, DELETE
Access-Control-Allow-Headers: X-Custom-Header
```

### 왜 Preflight가 필요한가?
- 보안 때문입니다.
- 웹 브라우저는 사용자의 정보를 보호하기 위해, 사용자가 모르는 사이에 민감한 요청이 전송되는 걸 방지하려고 합니다.
- 만약 Preflight 요청 없이 바로 민감한 API 요청을 허용하면:
  - 악성 사이트에서 사용자의 쿠키나 인증 정보가 자동 포함된 채로
  - 다른 사이트에 PUT, DELETE, POST 요청이 날아가
  - 정보 유출이나 데이터 삭제 같은 보안 문제가 생길 수 있습니다
- 따라서 Preflight 요청을 통해 서버가 명시적으로 허용하는지 확인한 후에만 본 요청을 진행합니다.

### 요약: Preflight = OPTIONS 요청
- 브라우저가 CORS 환경에서 **"이런 요청을 보내도 괜찮나요?"**라고 서버에 사전 확인할 때,
- HTTP OPTIONS 메서드를 사용해 Preflight 요청을 보냅니다.
- 즉, preflight 요청은 항상 다음과 같은 형식으로 전송됩니다:
```http
OPTIONS /some-resource HTTP/1.1
Origin: https://www.example.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: Content-Type
```
- 이 요청은 실제 데이터를 요청하는 것이 아니라,
- 서버가 "PUT 요청을 받아줄 의사가 있는지" 또는 "이 헤더들을 허용하는지" 확인하는 것입니다.

### 🔁 요청 흐름 예시
- 클라이언트에서 교차 오리진으로 PUT 요청을 하려고 함
- 브라우저가 먼저 아래와 같은 Preflight (OPTIONS) 요청을 전송함
```http
OPTIONS /api/data HTTP/1.1
Origin: https://www.client.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: Content-Type
```
- 서버가 이를 허용하는 경우:
```http
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://www.client.com
Access-Control-Allow-Methods: PUT
Access-Control-Allow-Headers: Content-Type
```
- 브라우저가 서버의 응답을 확인하고, 실제 PUT 요청을 보냄

### 💡 참고: Simple Request는 Preflight 없이
- 만약 요청이 아래 조건을 모두 만족하면 Preflight 없이 바로 요청이 전송됩니다:
  - 메서드가 GET, HEAD, POST 중 하나
  - Content-Type이 다음 중 하나:
    - text/plain
    - multipart/form-data
    - application/x-www-form-urlencoded
  - 커스텀 헤더 없음

## 📊 Amazon S3 액세스 로그와 미리 서명된 URL 이해하기

### 🔍 S3 액세스 로그 (Access Logs)

#### 액세스 로그란?
- S3 액세스 로그는 S3 버킷에 대한 **모든 요청(GET, PUT, DELETE 등)**을 기록하는 기능
- 요청이 승인되었는지, 거부되었는지와 상관없이 로그가 기록되며, 이 정보는 지정한 다른 S3 버킷에 로그 파일로 저장됨

#### 주요 특징
- 모든 계정의 요청을 로깅
- Athena, CloudWatch Logs, AWS Glue 등과 연동하여 분석 가능
- 로그는 특정 포맷으로 저장되며, 포맷 문서는 공식 문서 참고
- 로그 대상 버킷은 반드시 같은 리전 내에 있어야 함

#### 주의할 점
- 모니터링 대상 버킷과 로그 저장 버킷을 동일하게 설정하면 안 됨
- 이유는 다음과 같습니다:
  - 요청 → 로그 생성 → 로그 파일도 S3에 저장됨 → 그 저장도 로그로 인식됨 → 또 로그 생성…
  - 무한 로그 루프(Infinite Logging Loop) 발생
  - S3 버킷 용량 폭증 → 과금 증가 ⚠️
- 🧨 실제로 해보면 심각한 비용이 발생하므로 피해야 할 설정입니다.

### 🔑 미리 서명된 URL (Presigned URL)

#### 개요
- Amazon S3에서 미리 서명된 URL은 특정 객체에 대해 일시적으로 접근 권한을 부여할 수 있는 URL
- 이 URL을 가진 사람은 정해진 시간 동안 S3 객체에 접근할 수 있음

#### 생성 방식과 만료 시간
- S3 콘솔: 최대 12시간
- AWS CLI / SDK: 최대 168시간 (7일)

#### 사용자의 권한은 어떻게 되나요?
- 미리 서명된 URL은 URL을 생성한 사용자의 권한을 상속받음
- URL을 받은 사용자는 GET 또는 PUT 등 해당 작업을 수행할 수 있음

#### 📌 사용 사례 예시
| 시나리오 | 설명 |
|:--------:|:----:|
| 프라이빗 파일을 외부에 공유 | 퍼블릭으로 만들지 않고 특정 사람에게만 파일 접근 허용 |
| 로그인한 사용자만 다운로드 | 로그인 후 서버에서 presigned URL을 동적으로 생성 |
| 제한된 시간 동안 업로드 허용 | S3는 프라이빗 상태 유지, 특정 사용자만 일시적으로 업로드 가능 |
| 보안 유지 + 임시 권한 | 자격 증명 노출 없이 객체에 대한 임시 접근 제공 |

#### 🔁 동작 흐름 예시
- S3 버킷은 비공개 상태
- 버킷 소유자가 미리 서명된 URL 생성 (예: GET 방식, 1시간 유효)
- 해당 URL을 사용자에게 전달
- 사용자는 인증 없이 이 URL로 파일 다운로드
- URL이 만료되면 더 이상 접근 불가

## 🧩 마무리 요약

### 항목 | 설명
|:--------:|:----:|
| S3 액세스 로그 | 버킷에 대한 모든 요청을 별도 버킷에 기록 |
| 주의사항 | 로그 대상 버킷 = 모니터링 대상 버킷이면 무한 루프 발생 ⚠️ |
| Presigned URL | 사용자에게 임시로 파일 접근 권한 부여 |
| 보안성 | 버킷은 프라이빗 유지, URL에는 제한 시간과 작업 범위 명시 |
| 활용 예시 | 비공개 자료 공유, 인증된 사용자 전용 다운로드, 임시 업로드 허용 등 |

## 🔐 Amazon S3 액세스 포인트 (S3 Access Points)

### 왜 액세스 포인트가 필요한가?
- S3에 많은 데이터가 저장되어 있고, 여러 사용자나 부서(예: 재무팀, 영업팀, 분석팀)가 각각 다른 데이터에 접근해야 한다면, 버킷 정책이 점점 복잡해지고 관리하기 어려워집니다.
- 이를 해결하기 위해 Amazon은 **S3 액세스 포인트(Access Point)**라는 기능을 제공합니다.

### S3 액세스 포인트란?
- S3 액세스 포인트는 특정 사용자나 서비스가 S3 버킷의 일부 데이터에 정해진 규칙과 정책으로 접근할 수 있도록 만들어진 전용 엔드포인트
- 각 액세스 포인트에는 고유한 정책과 DNS 이름이 있으며, 기존의 복잡한 버킷 정책을 단순화하고 보안을 세분화할 수 있게 합니다.

### 예시 시나리오
- 하나의 S3 버킷에 다양한 데이터가 있음:
  - /finance/*: 재무 데이터
  - /sales/*: 영업 데이터
  - /analytics/*: 분석용 데이터
- 각 부서마다 다른 접근 권한을 부여하고자 함

### 접근 방식
| 액세스 포인트 이름 | 접근 대상 접두사 | 권한 범위 |
|:--------:|:----:|:----:|
| finance-ap | /finance/ | 읽기, 쓰기 |
| sales-ap | /sales/ | 읽기, 쓰기 |
| analytics-ap | /finance/, /sales/ | 읽기 전용 |
- 각 액세스 포인트에는 자체 정책을 연결할 수 있어, 버킷 정책을 복잡하게 만들지 않고도 세분화된 보안 관리가 가능합니다.

### 🔄 액세스 포인트의 동작 구조
- IAM 사용자가 finance-ap를 통해 S3에 접근
- 이 포인트는 finance/ 접두사만 허용
- sales/ 데이터에는 접근 불가
- analytics-ap는 읽기 전용으로 finance/와 sales/ 모두에 접근 가능
- ✔️ 이렇게 액세스 포인트를 나누면 권한을 역할/부서별로 명확히 구분할 수 있습니다.

## VPC와 액세스 포인트: 프라이빗 접근

### VPC 오리진이란?
- S3 액세스 포인트는 **VPC 오리진(VPC Origin)**을 설정할 수 있어 인터넷을 거치지 않고, 프라이빗하게 S3에 접근하도록 만들 수 있습니다.

### 사용 시나리오
- VPC 내의 EC2 인스턴스가 S3 데이터에 접근해야 할 때
- 민감한 데이터이기 때문에 인터넷을 통하지 않도록 해야 할 때

### 구성 요소
| 구성 요소 | 설명 |
|:--------:|:----:|
| VPC 엔드포인트 (Interface Endpoint) | VPC와 S3 Access Point를 연결하는 프라이빗 네트워크 경로 |
| VPC 엔드포인트 정책 | 어떤 S3 버킷과 액세스 포인트에 접근 가능한지 정의 |

### 보안 계층
- S3 액세스 포인트를 사용할 경우 다음의 3단계 보안 계층이 적용됩니다:
  - IAM 정책 – 사용자의 접근 제어
  - 액세스 포인트 정책 – 어떤 데이터(prefix)에 어떤 방식으로 접근 가능한지
  - 버킷 정책 – 버킷 차원의 추가 보안

## 요약
| 항목 | 설명 |
|:--------:|:----:|
| S3 Access Point | S3 버킷에 대한 세분화된 접근 경로 생성 |
| 정책 단순화 | 부서/사용자별로 분리된 정책을 적용할 수 있음 |
| 보안 관리 유연화 | 각각의 액세스 포인트에 고유 정책 부여 가능 |
| VPC 연동 | 인터넷 없이 EC2 → S3 접근 가능 (프라이빗) |
| VPC 엔드포인트 필요 | 프라이빗 연결을 위해 VPC Interface Endpoint 사용 |

## 🧠 Amazon S3 Object Lambda

### 📌 S3 Object Lambda란?
- S3 Object Lambda는 Amazon S3의 객체에 접근할 때, 사용자 정의 Lambda 함수로 객체 데이터를 동적으로 수정할 수 있도록 해주는 기능
- 즉, 객체를 수정한 새 버전의 파일을 미리 만들어 저장하지 않고, 요청 시점에 동적으로 변환 또는 필터링된 객체를 반환할 수 있습니다.

### 동작 방식

#### 구성 요소
- S3 버킷: 원본 데이터를 저장하는 장소
- S3 액세스 포인트: 특정 데이터에 접근을 정의하는 엔드포인트
- S3 Object Lambda Access Point: Lambda 함수를 통해 객체를 수정하여 전달하는 엔드포인트
- Lambda 함수: 객체에 대한 변환, 필터링, 보강 등의 작업을 실행

#### 🔄 흐름
1. 애플리케이션이 Object Lambda Access Point에 객체 요청
2. 해당 요청이 Lambda 함수로 전달됨
3. Lambda 함수가 원본 S3 버킷에서 객체를 가져와 코드를 실행 (예: 필터링, 변환)
4. 수정된 객체가 애플리케이션으로 반환

### 🧭 활용 예시
1️⃣ 데이터 필터링: 분석 애플리케이션
- 시나리오: 분석 앱에서는 일부 필드가 삭제된 객체만 필요
- 문제점: 이를 위해 버킷을 복제하고 데이터를 가공하는 건 비효율적
- 해결 방법:
  - S3 Object Lambda를 이용해 분석용 Access Point를 생성
  - Lambda 함수에서 객체를 필터링해 필요한 데이터만 제공

2️⃣ 데이터 보강: 마케팅 애플리케이션
- 시나리오: 마케팅 앱에서는 객체에 고객 충성도 데이터를 추가해야 함
- 해결 방법:
  - Lambda 함수에서 외부 DB에서 추가 데이터를 가져와 객체에 보강
  - 마케팅 앱은 Object Lambda Access Point를 통해 보강된 객체만 받음

3️⃣ 개인 식별 정보(PII) 마스킹
- 분석 또는 테스트 환경에서 실사용자의 PII 데이터를 제거하거나 마스킹
- Lambda 함수에서 민감 정보를 제거하거나 익명화

4️⃣ 형식 변환
- 객체가 XML 형식인데 클라이언트는 JSON을 원할 때
- Lambda 함수에서 형식을 변환 후 응답

5️⃣ 이미지 처리
- 요청 시점에 이미지 크기를 조정하거나
- 워터마크를 사용자별로 동적으로 삽입하는 경우

### 핵심 장점
| 장점 | 설명 |
|:--------:|:----:|
| 버킷 복제 불필요 | 변형된 데이터를 위해 별도 버킷이나 객체 사본 필요 없음 |
| 요청 기반 처리 | 요청 시점에 실시간으로 객체 수정 가능 |
| 유연한 보안 | IAM, 액세스 포인트 정책으로 세부 접근 제어 가능 |
| 고비용 작업 회피 | 스토리지 비용 절감, 객체 버전 관리 최소화 |

## 요약
| 항목 | 설명 |
|:--------:|:----:|
| S3 Object Lambda | Lambda 함수를 통해 객체 요청 시 데이터를 동적으로 변경 |
| 구성 요소 | S3 버킷 + 액세스 포인트 + Object Lambda 액세스 포인트 + Lambda 함수 |
| 사용 목적 | 필터링, 변환, 보강, 마스킹, 실시간 이미지 처리 등 |
| 보안 관리 | IAM 정책 + Access Point 정책으로 세분화 가능 |
| 사용 사례 | 분석, 마케팅, 보안, 멀티포맷 응답 등 |
