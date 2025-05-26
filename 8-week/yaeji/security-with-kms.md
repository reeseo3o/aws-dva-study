# AWS 보안 및 암호화 서비스 요약

## 1. 암호화 기본 개념

### 1.1. 전송 중 암호화 (Encryption in Transit)
- **정의**: 데이터가 네트워크를 통해 전송되기 전에 암호화되고, 수신 후에 복호화되는 방식입니다. (예: TLS/SSL, HTTPS)
- **목적**: 중간자 공격(Man-in-the-Middle Attack)으로부터 데이터를 보호하여, 의도된 수신자만 데이터를 해독할 수 있도록 합니다.
- **작동 방식**:
    1. 클라이언트가 서버로 데이터를 전송 시, TLS 인증서를 사용하여 데이터를 암호화합니다.
    2. 암호화된 데이터는 네트워크를 통해 전송됩니다.
    3. 중간 서버는 암호화된 데이터를 해독할 수 없습니다.
    4. 대상 서버만이 TLS 복호화 메커니즘을 사용하여 데이터를 해독합니다.

### 1.2. 서버 측 저장 중 암호화 (Server-Side Encryption at Rest)
- **정의**: 데이터가 서버에 저장되기 전에 서버 측에서 암호화되고, 클라이언트에게 전송되기 전에 복호화되는 방식입니다.
- **목적**: 저장된 데이터 자체를 보호합니다.
- **작동 방식**:
    1. 데이터가 서버에 수신되면, 서버는 데이터 키(Data Key)를 사용하여 데이터를 암호화합니다.
    2. 암호화된 데이터는 스토리지에 저장됩니다.
    3. 클라이언트가 데이터를 요청하면, 서버는 데이터 키를 사용하여 암호화된 데이터를 복호화합니다.
    4. 복호화된 데이터가 클라이언트에게 전송됩니다. (필요시 전송 중 암호화 적용)

### 1.3. 클라이언트 측 암호화 (Client-Side Encryption)
- **정의**: 데이터가 클라이언트 측에서 암호화 및 복호화되며, 서버는 암호화된 데이터만 처리하고 해독할 수 없습니다.
- **목적**: 서버를 신뢰할 수 없는 경우에도 데이터의 기밀성을 유지합니다. 봉투 암호화(Envelope Encryption) 메커니즘을 사용할 수 있습니다.
- **작동 방식**:
    1. 클라이언트는 데이터 키를 사용하여 로컬에서 데이터를 암호화합니다.
    2. 암호화된 데이터는 서버(스토리지 서비스 등)로 전송되어 저장됩니다. 서버는 이 데이터를 해독할 수 없습니다.
    3. 클라이언트가 데이터를 다시 가져올 때, 암호화된 데이터를 직접 수신합니다.
    4. 클라이언트는 로컬에 있는 데이터 키를 사용하여 데이터를 복호화합니다.

## 2. AWS Key Management Service (KMS)

KMS는 암호화 키를 쉽게 생성하고 제어할 수 있게 해주는 관리형 서비스입니다.

### 2.1. KMS의 장점
- **IAM 통합**: IAM을 통해 키에 대한 권한 부여 및 접근 제어가 용이합니다.
- **CloudTrail 감사**: 키 사용과 관련된 모든 API 호출을 CloudTrail을 통해 감사할 수 있습니다 (시험 출제 가능성 높음).
- **AWS 서비스 통합**: EBS, S3, RDS, SSM 등 대부분의 AWS 서비스와 원활하게 통합되어 저장 데이터 암호화에 사용됩니다.
- **직접 사용**: API, AWS CLI, SDK를 통해 민감 정보를 KMS 키로 직접 암호화하여 코드나 환경 변수에 안전하게 저장할 수 있습니다.

### 2.2. KMS 키 (구: 고객 마스터 키 - CMK)
- **대칭 KMS 키**:
    - 암호화와 복호화에 동일한 단일 키를 사용합니다.
    - KMS와 통합된 모든 AWS 서비스는 대칭 키를 사용합니다.
    - 키 자체에 직접 접근할 수 없으며, AWS API 호출을 통해서만 키를 활용할 수 있습니다.
- **비대칭 KMS 키**:
    - 공개 키(암호화용)와 개인 키(복호화용) 두 개의 키를 사용합니다.
    - 암호화/복호화 또는 서명/확인 작업에 사용됩니다.
    - 공개 키는 KMS에서 다운로드 가능하지만, 개인 키는 API 호출을 통해서만 접근 가능합니다.
    - 클라우드 외부에서 암호화를 수행하고, AWS 내에서 개인 키로 복호화하는 시나리오에 사용됩니다.

### 2.3. KMS 키의 종류
- **AWS 소유 키 (AWS Owned Keys)**:
    - 무료입니다.
    - 예: S3의 SSE-S3, DynamoDB 소유 키 등.
    - 사용자가 직접 보거나 관리할 수 없습니다.
- **AWS 관리형 키 (AWS Managed Keys)**:
    - 무료입니다.
    - 형식: `aws/<서비스명>` (예: `aws/rds`, `aws/ebs`).
    - 할당된 서비스 내에서만 사용 가능합니다.
    - 키 정책을 수정할 수 없습니다.
- **고객 관리형 키 (Customer Managed Keys - CMK)**:
    - 월 1달러의 비용이 발생합니다 (API 호출 비용 별도).
    - 사용자가 직접 생성, 관리하며 키 정책을 제어할 수 있습니다.
    - 키 자동 교체 설정, 키 가져오기(BYOK)가 가능합니다.

### 2.4. KMS 키 자동 교체
- **AWS 관리형 KMS 키**: 1년마다 자동으로 교체됩니다.
- **고객 관리형 KMS 키**: 1년마다 자동 교체를 활성화해야 합니다 (선택 사항).
- **가져온 KMS 키**: 수동으로만 교체 가능하며, 별칭(Alias)을 사용해야 합니다.

### 2.5. KMS 키의 리전 범위
- KMS 키는 리전별로 범위가 지정됩니다.
- 다른 리전으로 암호화된 리소스(예: EBS 스냅샷)를 복사하려면, 대상 리전의 다른 KMS 키로 재암호화해야 합니다. 동일한 KMS 키는 두 리전에 존재할 수 없습니다.

### 2.6. KMS 키 정책
- 키에 대한 접근을 제어하는 JSON 문서입니다. S3 버킷 정책과 유사합니다.
- 키 정책이 없으면 아무도 해당 키에 접근할 수 없습니다.
- **기본 키 정책**: 사용자 지정 키 정책이 제공되지 않을 때 생성되며, 계정의 모든 IAM 사용자/역할이 (IAM 정책에서 허용 시) 키에 접근할 수 있도록 합니다.
- **사용자 지정 키 정책**: 특정 사용자, 역할, 계정에게 키 사용 및 관리 권한을 세밀하게 부여할 수 있습니다. 교차 계정 접근 허용에 유용합니다.
    - 예시: 암호화된 스냅샷을 다른 계정과 공유 시, 소스 계정의 CMK 정책에 대상 계정의 접근을 허용하고, 대상 계정은 공유받은 스냅샷을 자신의 CMK로 재암호화하여 사용합니다.

### 2.7. KMS CLI 예제 (암호화 및 복호화)
1.  **파일 생성**: `ExampleSecretFile.txt` (내용: `SuperSecretPassword`)
2.  **암호화**:
    ```bash
    aws kms encrypt --key-id alias/tutorial --plaintext fileb://ExampleSecretFile.txt --query CiphertextBlob --output text --region eu-west-2 > ExampleSecretFileEncrypted.base64
    ```
    - `--key-id`: 사용할 KMS 키의 별칭, ID 또는 ARN.
    - `--plaintext`: 암호화할 파일.
    - 결과: Base64로 인코딩된 암호문.
3.  **Base64 디코드 (Linux/Mac)**:
    ```bash
    base64 --decode ExampleSecretFileEncrypted.base64 > ExampleSecretFileEncrypted
    ```
    - 결과: 바이너리 암호화 파일.
4.  **복호화**:
    ```bash
    aws kms decrypt --ciphertext-blob fileb://ExampleSecretFileEncrypted --query Plaintext --output text --region eu-west-2 > ExampleFileDecrypted.base64
    ```
    - KMS는 암호문(CiphertextBlob)에 포함된 메타데이터를 통해 어떤 키를 사용해야 할지 자동으로 압니다.
    - 결과: Base64로 인코딩된 평문.
5.  **Base64 디코드 (Linux/Mac)**:
    ```bash
    base64 --decode ExampleFileDecrypted.base64 > ExampleFileDecrypted.txt
    ```
    - 결과: 원본 평문 파일 (`SuperSecretPassword`).

### 2.8. KMS API와 봉투 암호화 (Envelope Encryption)

#### 2.8.1. KMS 암호화/복호화 API (4KB 이하 데이터)
- **암호화 (`Encrypt` API)**:
    1. 평문 데이터(4KB 이하)와 사용할 CMK ID를 KMS로 전송합니다.
    2. KMS는 IAM 권한을 확인합니다.
    3. 권한이 있으면 데이터를 암호화하여 암호문(Ciphertext)을 반환합니다.
- **복호화 (`Decrypt` API)**:
    1. 암호문을 KMS로 전송합니다. (KMS는 암호문에 포함된 정보로 어떤 CMK가 사용되었는지 자동으로 식별)
    2. KMS는 IAM 권한을 확인합니다.
    3. 권한이 있으면 데이터를 복호화하여 평문을 반환합니다.

#### 2.8.2. 봉투 암호화 (4KB 초과 데이터)
- **목적**: KMS의 4KB 크기 제한을 넘어 큰 데이터를 효율적으로 암호화하기 위함입니다. 데이터 암호화는 클라이언트 측에서 수행하고, 데이터 암호화에 사용된 키(데이터 키, DEK)만 KMS를 통해 보호합니다.
- **주요 API**: `GenerateDataKey`
- **작동 원리 (암호화)**:
    1. 클라이언트는 KMS에 `GenerateDataKey` API를 호출하며 사용할 CMK ID를 지정합니다.
    2. KMS는 IAM 권한 확인 후, 다음 두 가지를 생성하여 반환합니다:
        - 평문 데이터 키 (Plaintext Data Key, DEK)
        - CMK로 암호화된 데이터 키 (Encrypted Data Key)
    3. 클라이언트는 평문 DEK를 사용하여 로컬에서 큰 파일을 암호화합니다. (CPU 사용)
    4. 클라이언트는 암호화된 파일과 암호화된 DEK를 함께 저장하거나 전송합니다. (이것이 "봉투")
- **작동 원리 (복호화)**:
    1. 클라이언트는 저장된 "봉투"에서 암호화된 파일과 암호화된 DEK를 가져옵니다.
    2. 클라이언트는 암호화된 DEK를 KMS의 `Decrypt` API로 전송하여 복호화를 요청합니다.
    3. KMS는 IAM 권한 확인 후, 암호화된 DEK를 CMK로 복호화하여 평문 DEK를 반환합니다.
    4. 클라이언트는 반환받은 평문 DEK를 사용하여 로컬에서 암호화된 파일을 복호화합니다.

#### 2.8.3. AWS Encryption SDK
- 봉투 암호화 과정을 추상화하여 쉽게 구현할 수 있도록 돕는 SDK입니다. (Java, Python, C, JavaScript 등 지원)
- **데이터 키 캐싱 (Data Key Caching)**:
    - 동일한 데이터 키를 재사용하여 KMS API 호출 수를 줄이고 비용을 절감합니다.
    - 보안 수준이 다소 낮아질 수 있으므로 (하나의 데이터 키가 여러 파일에 사용됨), API 호출과 보안 간의 트레이드오프를 고려해야 합니다.
    - `LocalCryptoMaterialsCache`를 사용하여 캐시 크기, 키 지속 시간, 암호화할 최대 용량/메시지 수 등을 설정할 수 있습니다.

#### 2.8.4. 주요 KMS 대칭 API 요약 (시험 대비)
- **`Encrypt`**: 4KB 이하 데이터 암호화.
- **`Decrypt`**: 4KB 이하 데이터 복호화. 봉투 암호화 시 암호화된 데이터 키(DEK) 복호화에도 사용.
- **`GenerateDataKey`**: 고유한 대칭 데이터 키(DEK)를 생성. 평문 DEK와 지정된 CMK로 암호화된 DEK를 모두 반환. 봉투 암호화에 사용.
- **`GenerateDataKeyWithoutPlaintext`**: DEK를 생성하지만 평문은 반환하지 않고, 지정된 CMK로 암호화된 DEK만 반환. 나중에 복호화 필요.
- **`GenerateRandom`**: 무작위 바이트 문자열 생성.

### 2.9. AWS Encryption CLI
- `pip install aws-encryption-sdk-cli`로 설치.
- 봉투 암호화를 사용하여 로컬 파일을 암호화/복호화하는 CLI 도구.
- 사용 예:
    ```bash
    # 환경 변수 설정 (사용할 CMK의 ARN)
    export key="arn:aws:kms:eu-west-2:ACCOUNT_ID:key/YOUR_KEY_ID"

    # 파일 생성
    echo "This is a secret message larger than 1MB..." > hello.txt

    # 암호화
    aws-encryption-cli --encrypt --input hello.txt --master-keys key=$key --metadata-output ~/metadata --output .
    # --output . : 현재 디렉토리에 hello.txt.encrypted 파일 생성

    # 복호화
    aws-encryption-cli --decrypt --input hello.txt.encrypted --master-keys key=$key --metadata-output ~/metadata_dec --output .
    # --output . : 현재 디렉토리에 hello.txt.encrypted.decrypted 파일 생성
    ```
- 메타데이터 파일에는 암호화에 사용된 알고리즘, 데이터 키 정보 등이 JSON 형식으로 저장됩니다.

### 2.10. KMS 요청 할당량 (Request Quotas) 및 조절 (Throttling)
- KMS API 호출에는 할당량(제한)이 있으며, 초과 시 `ThrottlingException` (HTTP 400)이 발생합니다.
- **대처 방안**:
    1.  **지수 백오프 (Exponential Backoff)**: API 호출 실패 시 재시도 간격을 점차 늘려가며 재시도합니다.
    2.  **데이터 키 캐싱 (DEK Caching)**: AWS Encryption SDK 사용 시, 데이터 키를 로컬에 캐시하여 KMS API 호출 수를 줄입니다.
    3.  **할당량 증가 요청**: AWS 고객센터를 통해 서비스 할당량 증가를 요청합니다.
- 모든 암호화 작업 (Encrypt, Decrypt, GenerateDataKey, GenerateRandom 등)은 할당량을 공유합니다.
- 할당량은 리전별로 다르며, 계정 내 모든 서비스에서 공유됩니다. (예: 초당 5,500 ~ 30,000회)

## 3. Lambda와 KMS 통합 (환경 변수 암호화)

Lambda 함수의 환경 변수에 저장된 민감 정보(예: DB 비밀번호)를 KMS를 사용하여 암호화할 수 있습니다.

1.  **문제점**:
    -   코드에 비밀번호를 하드코딩: 보안에 매우 취약.
    -   환경 변수에 평문으로 저장: Lambda 함수 설정에 접근 가능한 사용자가 비밀번호를 볼 수 있음.
2.  **해결책**: Lambda 환경 변수 암호화 기능 사용.
    -   Lambda 함수 설정 -> 환경 변수 -> "암호화 구성"에서 "전송 중 암호화를 위한 도우미 활성화" 체크.
    -   사용할 KMS 키(고객 관리형 키 권장)를 선택하고 "암호화" 버튼 클릭.
    -   Lambda는 암호화된 환경 변수 값을 저장하고, 복호화를 위한 코드 스니펫을 제공합니다.
3.  **Lambda 함수 코드 (Python 예시)**:
    ```python
    import boto3
    import os

    ENCRYPTED_DB_PASSWORD = os.environ['DB_PASSWORD']
    # KMS를 사용하여 복호화
    DECRYPTED_DB_PASSWORD = boto3.client('kms').decrypt(
        CiphertextBlob=bytes.fromhex(ENCRYPTED_DB_PASSWORD), # 또는 base64 디코딩
        EncryptionContext={'LambdaFunctionName': os.environ['AWS_LAMBDA_FUNCTION_NAME']}
    )['Plaintext'].decode('utf-8')

    def lambda_handler(event, context):
        print("Encrypted: " + ENCRYPTED_DB_PASSWORD)
        print("Decrypted: " + DECRYPTED_DB_PASSWORD)
        # DECRYPTED_DB_PASSWORD를 사용하여 DB 연결 등 수행
        return "Success"
    ```
    *참고: AWS Lambda 콘솔에서 제공하는 코드 스니펫은 base64 인코딩된 암호문을 가정할 수 있습니다. `CiphertextBlob` 전달 시 적절한 디코딩(예: `base64.b64decode()`)이 필요할 수 있습니다. 위의 예제는 일반적인 KMS `decrypt` 호출을 보여줍니다. Lambda 환경 변수 암호화의 실제 구현은 `bytes.fromhex()` 대신 다른 방식을 사용할 수 있습니다.*
4.  **IAM 권한**: Lambda 실행 역할(Execution Role)에 다음 권한이 필요합니다.
    -   선택한 KMS 키에 대한 `kms:Decrypt` 권한.
    -   (선택 사항) 환경 변수 암호화 시 Lambda 서비스가 내부적으로 사용하는 KMS 키에 대한 권한 (일반적으로 자동 처리됨).
5.  **장점**: 코드나 Lambda 구성에서 평문 비밀번호가 노출되지 않으며, KMS 키에 대한 접근 권한이 있는 경우에만 복호화 가능.

## 4. S3와 KMS 통합 (S3 버킷 키)

SSE-KMS를 사용할 때 S3 버킷 키(Bucket Key)를 활성화하면 KMS API 호출량을 최대 99%까지 줄여 비용을 절감할 수 있습니다.

- **작동 원리**:
    1.  S3는 사용자가 지정한 CMK(고객 마스터 키)를 사용하여 특정 S3 버킷에 대한 단기적인 "S3 버킷 키"를 생성합니다. 이 작업은 KMS를 호출합니다.
    2.  이 S3 버킷 키는 S3 내에 잠시 캐시됩니다.
    3.  버킷에 업로드되는 객체들은 이 S3 버킷 키를 사용하여 봉투 암호화 방식으로 암호화됩니다. 즉, 각 객체에 대한 데이터 키가 S3 버킷 키로 암호화됩니다. 이 과정에서는 KMS를 직접 호출하지 않습니다.
    4.  S3 버킷 키는 주기적으로 순환됩니다.
- **효과**:
    -   KMS API 호출 수 감소 -> KMS 비용 절감.
    -   KMS 요청 할당량 초과 위험 감소.
    -   CloudTrail 내 KMS 관련 이벤트 수 감소.
- **설정**: S3 버킷 생성 또는 속성 편집 시, 기본 서버 측 암호화 설정에서 SSE-KMS를 선택하고 "버킷 키"를 "활성화"(기본값)로 설정합니다.

## 4.1. S3 SSE-C (Server-Side Encryption with Customer-Provided Keys)

SSE-C는 고객이 직접 제공한 암호화 키를 사용하여 S3에서 서버 측 암호화를 수행하는 방식입니다.

### 4.1.1. SSE-C 주요 특징
- **키 관리 책임**: 암호화 키는 고객이 직접 제공하고 관리해야 합니다.
- **암호화 방식**: Amazon S3는 제공된 키로 AES-256 암호화를 수행합니다.
- **키 저장 방식**: S3는 암호화 키를 저장하지 않고, 요청이 처리된 후 메모리에서 즉시 삭제합니다.
- **HMAC 저장**: S3는 제공된 키의 HMAC(Hash-based Message Authentication Code) 값만 저장합니다.

### 4.1.2. SSE-C 작동 방식
1. **업로드 시**:
   - 클라이언트가 객체와 함께 암호화 키를 HTTPS를 통해 S3에 전송
   - S3는 제공된 키로 객체를 암호화
   - S3는 키의 HMAC 값을 계산하여 저장
   - 원본 암호화 키는 메모리에서 즉시 삭제

2. **다운로드 시**:
   - 클라이언트가 객체 요청과 함께 암호화 키를 HTTPS를 통해 전송
   - S3는 제공된 키의 HMAC 값을 계산하여 저장된 HMAC 값과 비교
   - HMAC 값이 일치하면 객체를 복호화하여 반환
   - 키가 일치하지 않으면 403 Forbidden 오류 반환

---

#### [시험 대비] SSE-C 업로드 시 필수 헤더 및 오답 해설

SSE-C(Server-Side Encryption with Customer-Provided Keys) 방식으로 S3에 객체를 업로드할 때 반드시 아래 3가지 헤더를 요청에 포함해야 합니다.

- `x-amz-server-side-encryption-customer-algorithm`: 암호화 알고리즘 지정 (항상 "AES256")
- `x-amz-server-side-encryption-customer-key`: base64로 인코딩된 256비트 암호화 키
- `x-amz-server-side-encryption-customer-key-MD5`: 암호화 키의 base64 인코딩된 MD5 다이제스트 (무결성 검증용)

이 세 가지 헤더가 모두 필수입니다. 하나라도 빠지면 업로드가 거부됩니다.

**각 헤더의 역할**
- `x-amz-server-side-encryption-customer-algorithm`: S3에 암호화 알고리즘을 명시적으로 전달 (SSE-C는 반드시 AES256)
- `x-amz-server-side-encryption-customer-key`: 실제 암호화에 사용할 키 (base64 인코딩)
- `x-amz-server-side-encryption-customer-key-MD5`: 키의 무결성 검증을 위한 MD5 다이제스트 (base64 인코딩)

**시험에 자주 출제되는 오답 보기 해설**
- `x-amz-server-side-encryption` 또는 `x-amz-server-side-encryption-aws-kms-key-id`는 SSE-KMS(즉, AWS KMS 키 사용)에서만 사용하며, SSE-C에는 적용되지 않음
- `x-amz-server-side-encryption-customer-key`만 단독 사용은 불가 (알고리즘, MD5 모두 필수)
- SSE-C는 반드시 위 3가지 헤더를 모두 포함해야 하며, 키 분실 시 복구 불가

**실제 업로드 예시 (Python - boto3)**
```python
s3.put_object(
    Bucket='your-bucket',
    Key='encrypted-file.txt',
    Body=b'Your data here',
    SSECustomerKey=encryption_key,  # x-amz-server-side-encryption-customer-key
    SSECustomerAlgorithm='AES256',  # x-amz-server-side-encryption-customer-algorithm
    SSECustomerKeyMD5=md5_digest    # x-amz-server-side-encryption-customer-key-MD5
)
```

---

## 부록: S3 SSE-KMS 대용량 업로드 권한 문제

### 문제 상황 요약
- aws s3 cp로 100GB 이상 파일을 SSE-KMS가 활성화된 S3 버킷에 업로드 시 Access Denied 오류 발생
- 작은 파일은 정상적으로 업로드됨

### 원인 분석
1. **Multipart Upload**
   - AWS CLI는 8MB 이상 파일에 대해 자동으로 multipart upload를 사용
   - multipart upload는 각 파트별로 암호화/키 관리가 개별적으로 수행됨
   - 업로드 완료 시 S3가 각 파트를 조합하며 KMS에 복호화(kms:Decrypt) 권한이 필요
2. **kms:Decrypt 권한 부족**
   - 작은 파일(단일 put)은 kms:GenerateDataKey만 필요
   - multipart upload는 완료 시 kms:Decrypt 권한까지 필요
   - 해당 권한이 없으면 Access Denied 발생

### 관련 시험 문제 예시

#### 문제 1
SSE-KMS가 활성화된 S3 버킷에 100GB 파일을 AWS CLI로 업로드할 때 Access Denied 오류가 발생했다. 작은 파일은 정상적으로 업로드된다. 가장 가능성 높은 원인은?

A) KMS의 최대 암호화 크기 제한(4KB) 때문이다.
B) IAM 정책에 100GB 이상 업로드 제한이 걸려 있다.
C) multipart upload 완료 시 kms:Decrypt 권한이 없기 때문이다.
D) kms:Encrypt 권한이 없기 때문이다.

**정답:** C
**해설:** multipart upload는 완료 시 S3가 각 파트를 검증하기 위해 kms:Decrypt 권한이 필요하다. 작은 파일은 단일 put이므로 kms:GenerateDataKey만 필요하다.

#### 문제 2
SSE-KMS가 적용된 S3 버킷에 대용량 파일을 업로드할 때 필요한 최소 KMS 권한 조합은?

A) kms:GenerateDataKey
B) kms:GenerateDataKey, kms:Decrypt
C) kms:Encrypt, kms:Decrypt
D) kms:Encrypt

**정답:** B
**해설:** multipart upload는 각 파트 암호화에 kms:GenerateDataKey, 완료 시 kms:Decrypt 권한이 필요하다.

#### 문제 3
다음 중 S3 SSE-KMS 버킷에 대용량 파일 업로드 시 Access Denied 오류의 잘못된 원인 설명은?

A) KMS의 직접 암호화 크기 제한(4KB) 때문이다.
B) kms:Decrypt 권한이 없기 때문이다.
C) multipart upload가 사용되기 때문이다.
D) IAM 정책으로 파일 크기 제한이 걸려 있기 때문이다.

**정답:** A, D
**해설:**
A: S3는 자체적으로 대칭키로 암호화하므로 KMS의 4KB 제한과 무관하다.
D: IAM 정책은 파일 크기 기준 제한을 지원하지 않는다.

### 요점 정리
- multipart upload: AWS CLI는 8MB 이상 파일에 대해 자동으로 multipart upload 사용
- 각 파트별로 암호화/키 관리, 업로드 완료 시 kms:Decrypt 권한 필요
- 작은 파일(단일 put): kms:GenerateDataKey만 필요
- 대용량 파일(multipart): kms:GenerateDataKey + kms:Decrypt 필요
- kms:Encrypt는 직접 암호화할 때만 필요, S3는 자체적으로 처리
- KMS의 4KB 제한은 S3에 적용되지 않음
- IAM 정책으로 파일 크기 제한 불가
- kms:Encrypt 권한은 multipart upload와 직접적 관련 없음

---



