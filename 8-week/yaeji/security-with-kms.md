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

## 5. CloudHSM (Hardware Security Module)

CloudHSM은 전용 하드웨어 보안 모듈(HSM)을 제공하여 사용자가 암호화 키를 완벽하게 제어할 수 있도록 하는 서비스입니다.

### 5.1. CloudHSM 특징
- **전용 하드웨어**: FIPS 140-2 Level 3 규정을 준수하는 변조 방지 HSM 장치를 AWS 클라우드 내에 프로비저닝합니다.
- **키 완전 제어**: 사용자가 암호화 키를 직접 생성, 관리, 사용합니다. AWS는 HSM 장치에 접근할 수 없습니다.
- **지원 키 유형**: 대칭 및 비대칭 암호화 키 (SSL/TLS 키 등).
- **비용**: 프리 티어 없음. HSM 장치 사용 시간에 따라 비용 발생.
- **클라이언트 소프트웨어**: HSM 장치와 상호작용하기 위해 전용 클라이언트 소프트웨어 필요.
- **고가용성**: CloudHSM 클러스터는 여러 AZ에 분산되어 고가용성을 제공합니다.
- **IAM 권한**: HSM 클러스터 생성, 읽기, 업데이트, 삭제 등 높은 수준의 관리 작업에 사용. 키 관리 및 사용자 접근 권한은 CloudHSM 소프트웨어 내에서 별도로 관리.

### 5.2. KMS와의 비교
| 특징             | KMS                                  | CloudHSM                                  |
| ---------------- | ------------------------------------ | ----------------------------------------- |
| 테넌시           | 다중 테넌트 (Multi-tenant)             | 단일 테넌트 (Single-tenant, 전용 하드웨어) |
| 키 관리 주체     | AWS (소프트웨어 기반) / 사용자 (CMK) | 사용자 (하드웨어 기반, AWS 접근 불가)      |
| 마스터 키 종류   | AWS 소유, AWS 관리, 고객 관리형 CMK  | 고객 관리형 CMK만 지원                    |
| 키 접근성        | 여러 리전 (재암호화 필요)              | VPC 내 배포 (VPC 피어링으로 공유 가능)      |
| 암호화 가속      | 설정 불가                            | SSL/TLS 가속, Oracle TDE 가속 등 가능     |
| 접근 및 인증     | IAM                                  | 자체 보안 메커니즘 (HSM 사용자/권한 관리)   |
| 고가용성         | 관리형 서비스, 항상 사용 가능          | 다중 AZ에 걸친 HSM 장치 클러스터           |
| 비용             | 프리 티어 포함, 저렴                  | 프리 티어 없음, 상대적으로 고가             |

### 5.3. CloudHSM과 KMS 통합 (사용자 지정 키 스토어)
- **목적**: EBS, S3, RDS 등 AWS 서비스에서 CloudHSM의 키를 KMS 인터페이스를 통해 사용하기 위함입니다.
- **작동 방식**:
    1.  CloudHSM 클러스터를 생성합니다.
    2.  KMS에서 "사용자 지정 키 스토어(Custom Key Store)"를 생성하고, 이를 기존 CloudHSM 클러스터에 연결합니다.
    3.  AWS 서비스(예: EBS)에서 암호화 시, 이 사용자 지정 키 스토어에 연결된 KMS 키(CMK)를 선택합니다.
    4.  실제 암호화 작업은 CloudHSM 클러스터 내의 키를 활용하여 수행되지만, 모든 API 호출은 KMS를 통해 이루어지므로 CloudTrail에 기록됩니다.
- **이점**:
    -   CloudHSM의 강력한 보안 및 키 제어 기능을 활용.
    -   KMS의 편리한 서비스 통합 및 감사 기능(CloudTrail)을 동시에 사용.

## 6. SSM Parameter Store (Systems Manager Parameter Store)

구성 데이터 및 암호(비밀)를 안전하게 저장하고 관리하기 위한 서비스입니다.

### 6.1. Parameter Store 특징
- **안전한 저장**: KMS를 사용하여 파라미터 값을 암호화할 수 있습니다 (SecureString 유형).
- **계층 구조**: 경로 기반으로 파라미터를 구성하여 체계적인 관리가 가능합니다 (예: `/my-app/dev/db-url`). IAM 정책을 통해 경로별 접근 제어가 용이합니다.
- **버전 관리**: 파라미터 업데이트 시마다 버전이 생성되어 변경 이력 추적이 가능합니다.
- **IAM 통합**: IAM을 통해 파라미터에 대한 접근 권한을 제어합니다.
- **EventBridge 통합**: 파라미터 변경 또는 만료 시 알림을 받거나 자동화 작업을 트리거할 수 있습니다 (고급 티어).
- **CloudFormation 통합**: CloudFormation 템플릿에서 파라미터 값을 동적으로 참조할 수 있습니다.
- **무료 (표준 티어)**: 표준 파라미터는 무료로 사용할 수 있습니다 (고급 파라미터는 유료).

### 6.2. 파라미터 유형
- **`String`**: 일반 텍스트 문자열.
- **`StringList`**: 쉼표로 구분된 문자열 목록.
- **`SecureString`**: KMS를 사용하여 암호화되는 문자열. 암호, API 키 등 민감 정보 저장에 사용.

### 6.3. 파라미터 티어
- **표준 (Standard) 티어**:
    -   파라미터 크기: 최대 4KB.
    -   파라미터 정책: 사용 불가.
    -   비용: 무료.
    -   계정당 파라미터 수: 10,000개.
- **고급 (Advanced) 티어**:
    -   파라미터 크기: 최대 8KB.
    -   파라미터 정책: 사용 가능 (만료 알림, 변경 없음 알림 등).
    -   비용: 파라미터당 월 $0.05 (API 사용량 별도).
    -   계정당 파라미터 수: 100,000개.

### 6.4. 파라미터 정책 (고급 티어 전용)
- 파라미터의 만료 날짜 설정, 특정 기간 변경 없을 시 알림 등의 규칙을 적용할 수 있습니다.
- 예:
    -   **만료 정책**: 지정된 시간이 되면 파라미터 삭제 (또는 삭제 전 알림).
    -   **변경 없음 알림 정책**: 일정 기간 동안 파라미터가 갱신되지 않으면 알림.

### 6.5. CLI 및 SDK 사용 예 (Python - Boto3)
- **파라미터 가져오기**:
    ```python
    import boto3
    ssm = boto3.client('ssm', region_name='your-region')

    # 단일 파라미터 (SecureString의 경우 복호화 필요)
    response = ssm.get_parameter(Name='/my-app/dev/db-password', WithDecryption=True)
    password = response['Parameter']['Value']

    # 경로별 파라미터 가져오기
    response = ssm.get_parameters_by_path(Path='/my-app/dev/', Recursive=True, WithDecryption=True)
    for param in response['Parameters']:
        print(f"{param['Name']}: {param['Value']}")
    ```
- **Lambda와 연동**: 환경 변수(예: `DEV_OR_PROD`)를 사용하여 동적으로 파라미터 경로를 구성하고, Lambda 실행 역할에 SSM 접근 권한 및 (SecureString의 경우) KMS 복호화 권한을 부여합니다.

### 6.6. NoChangeNotification 정책과 EventBridge 통합

NoChangeNotification 정책은 매개변수가 일정 기간 동안 변경되지 않았을 때 알림을 발생시키는 고급 티어 전용 기능입니다.

#### 6.6.1. 구현 단계
1. **고급 티어로 업그레이드**:
    - 표준 티어에서는 정책 설정이 불가능합니다.
    - 고급 티어로 업그레이드해야 NoChangeNotification 정책을 사용할 수 있습니다.
    - AWS CLI 명령어:
    ```bash
    aws ssm update-parameter --name "/my-app/prod/db-url" --tier Advanced
    ```

2. **NoChangeNotification 정책 추가**:
    - 매개변수의 LastModifiedTime을 기준으로 동작합니다.
    - 지정된 기간(예: 90일) 동안 변경이 없으면 EventBridge 이벤트가 발생합니다.
    - AWS CLI 명령어:
    ```bash
    aws ssm put-parameter --name "/my-app/prod/db-url" --value "your-value" --type String --policies '[{"Type": "NoChangeNotification","Attributes": {"After": "90"}}]'
    ```

3. **EventBridge 규칙 설정**:
    ```json
    {
      "source": ["aws.ssm"],
      "detail-type": ["Parameter Store Policy Action"],
      "detail": {
        "operation": ["NoChangeNotification"],
        "name": ["/my-app/prod/db-url"]
      }
    }
    ```

4. **SNS 주제 생성 및 구독**:
    - EventBridge 규칙의 대상으로 SNS 주제를 지정합니다.
    - SNS 주제에 이메일, SMS 등의 구독을 추가하여 알림을 받습니다.

#### 6.6.2. 주의사항
- **티어 제한**: NoChangeNotification 정책은 고급 티어에서만 사용 가능합니다.
- **비용**: 고급 티어는 매개변수당 월 $0.05의 비용이 발생합니다.
- **정책 조합**: 여러 정책(예: ExpirationNotification + NoChangeNotification)을 동시에 적용할 수 있습니다.
- **EventBridge 규칙**: 정책이 없으면 이벤트가 발생하지 않으므로, EventBridge 규칙만 설정하는 것은 의미가 없습니다.

#### 6.6.3. 사용 예시 (Python - Boto3)
```python
import boto3

ssm = boto3.client('ssm')

# 매개변수를 고급 티어로 업그레이드하고 NoChangeNotification 정책 설정
response = ssm.put_parameter(
    Name='/my-app/prod/db-url',
    Value='your-value',
    Type='String',
    Tier='Advanced',
    Policies='[{"Type": "NoChangeNotification","Attributes": {"After": "90"}}]',
    Overwrite=True
)

# 정책 확인
response = ssm.get_parameter(
    Name='/my-app/prod/db-url'
)
print(f"Parameter Version: {response['Parameter']['Version']}")
```

## 7. AWS Secrets Manager

암호, API 키, 데이터베이스 자격 증명 등 비밀 정보를 안전하게 저장, 관리, 순환하기 위한 서비스입니다. SSM Parameter Store보다 비밀 관리에 특화된 기능을 제공합니다.

### 7.1. Secrets Manager 특징
- **자동 암호 순환**: Lambda 함수를 사용하여 정의된 일정(예: 매 30일)에 따라 비밀을 자동으로 순환할 수 있습니다.
    -   RDS (MySQL, PostgreSQL, Aurora 등), Redshift, DocumentDB 등과 강력하게 통합되어, AWS가 제공하는 Lambda 함수를 통해 해당 서비스의 자격 증명을 자동으로 순환합니다.
    -   기타 비밀(예: API 키)의 경우 사용자가 직접 순환 로직을 담은 Lambda 함수를 작성해야 합니다.
- **KMS 암호화**: 저장되는 모든 비밀은 KMS를 사용하여 암호화됩니다 (필수).
- **IAM 통합**: IAM을 통해 비밀에 대한 접근 권한을 제어합니다.
- **CloudFormation 통합**: CloudFormation 템플릿에서 비밀 값을 동적으로 참조할 수 있습니다.
- **멀티리전 비밀**: 비밀을 여러 AWS 리전에 복제하여 고가용성 및 재해 복구 전략을 지원합니다. 기본 리전에 문제가 생기면 복제본 비밀을 독립된 비밀로 승격시킬 수 있습니다.
- **비용**: 암호당 월 $0.40, API 호출 10,000회당 $0.05 (30일 무료 평가판 제공).

### 7.2. SSM Parameter Store와의 차이점

| 특징             | Secrets Manager                                     | SSM Parameter Store                                       |
| ---------------- | --------------------------------------------------- | --------------------------------------------------------- |
| 주요 용도        | 비밀(암호, API 키, DB 자격증명) 관리 및 자동 순환    | 구성 데이터 및 비밀 저장 (수동 순환 또는 외부 자동화 필요) |
| 자동 순환        | 내장 기능 (Lambda 연동)                             | 내장 기능 없음 (CloudWatch Events + Lambda로 직접 구현)     |
| 비용             | 상대적으로 높음 (암호당 과금)                         | 저렴 (표준 티어 무료, 고급 티어 유료)                      |
| KMS 암호화       | 필수                                                | 선택 사항 (`SecureString` 유형에만 적용)                  |
| 서비스 통합      | RDS, DocumentDB 등 DB 자격 증명 순환에 특화된 통합 | 범용적                                                    |
| 멀티리전         | 멀티리전 비밀 기능 제공                             | 직접 구성 필요                                            |

### 7.3. 사용 예
- **비밀 저장**:
    -   RDS 데이터베이스 자격 증명, 다른 데이터베이스 자격 증명, 기타 유형의 비밀(키/값 쌍 또는 평문 JSON) 저장 가능.
    -   저장 시 사용할 KMS 키 선택.
- **자동 순환 설정**:
    -   순환 주기(예: 60일) 및 순환을 수행할 Lambda 함수 지정.
- **비밀 조회 (Python - Boto3)**:
    ```python
    import boto3
    client = boto3.client('secretsmanager', region_name='your-region')
    response = client.get_secret_value(SecretId='/prod/my-secret-api')
    # 비밀 값은 'SecretString' (텍스트) 또는 'SecretBinary' (바이너리)에 있음
    if 'SecretString' in response:
        secret = response['SecretString']
    else:
        secret = base64.b64decode(response['SecretBinary'])
    # secret은 JSON 문자열일 수 있으므로 파싱 필요
    # import json; secret_data = json.loads(secret)
    ```

## 8. CloudFormation Dynamic References

CloudFormation 템플릿 실행 시점에 SSM Parameter Store 또는 Secrets Manager에서 실제 값을 동적으로 가져와서 리소스 설정에 사용하는 기능입니다. 템플릿에 민감 정보를 직접 하드코딩하는 것을 방지합니다.

- **지원 방식**:
    -   `resolve:ssm:<parameter-name>:<version>`: SSM Parameter Store의 평문(`String`, `StringList`) 파라미터 값을 가져옵니다. 버전은 선택 사항.
    -   `resolve:ssm-secure:<parameter-name>:<version>`: SSM Parameter Store의 암호화된(`SecureString`) 파라미터 값을 가져옵니다. 버전은 선택 사항.
    -   `resolve:secretsmanager:<secret-id>:<secret-key>:<version-stage>:<version-id>`: Secrets Manager의 비밀 값을 가져옵니다.
        -   `<secret-id>`: 비밀의 이름 또는 ARN.
        -   `<secret-key>`: (선택 사항) JSON 구조의 비밀에서 특정 키의 값만 가져올 때 사용.
        -   `<version-stage>`: (선택 사항) 특정 버전 스테이지(예: `AWSCURRENT`).
        -   `<version-id>`: (선택 사항) 특정 버전 ID.
- **예시 (Secrets Manager에서 RDS 마스터 사용자 암호 가져오기)**:
    ```yaml
    Resources:
      MyRDSInstance:
        Type: AWS::RDS::DBInstance
        Properties:
          MasterUsername: MyUser
          MasterUserPassword: "{{resolve:secretsmanager:MyRDSPasswordSecret:SecretString:AWSCURRENT}}"
          # ... 기타 속성 ...
    ```
- **RDS와 Secrets Manager 통합 (CloudFormation)**:
    1.  **RDS가 비밀 관리**: `AWS::RDS::DBCluster` 또는 `AWS::RDS::DBInstance` 리소스에서 `ManageMasterUserPassword: true`로 설정하면, RDS가 Secrets Manager에 마스터 사용자 암호를 자동으로 생성하고 관리(자동 순환 포함)합니다. 생성된 비밀의 ARN은 `Fn::GetAtt`로 참조 가능.
    2.  **CloudFormation이 비밀 생성, RDS가 참조**: CloudFormation 템플릿 내에서 `AWS::SecretsManager::Secret` 리소스를 사용하여 비밀을 직접 생성 (예: `GenerateSecretString` 사용). RDS 인스턴스는 이 비밀을 동적 참조로 사용. 비밀 순환은 `AWS::SecretsManager::RotationSchedule` 및 `AWS::SecretsManager::ResourcePolicy` 와 `AWS::RDS::DBInstance` (또는 `DBCluster`)의 `SecretRotation` 속성을 통해 설정 필요.

## 9. CloudWatch Logs 암호화

CloudWatch Logs 로그 그룹의 데이터를 KMS를 사용하여 암호화할 수 있습니다.

- **암호화 단위**: 로그 그룹 레벨에서 수행 (로그 스트림 레벨 아님).
- **설정 방법**:
    -   CloudWatch 콘솔에서는 직접 CMK 연결 불가.
    -   AWS CLI 또는 SDK를 통해 CloudWatch Logs API 사용.
    -   **명령어**:
        -   `aws logs associate-kms-key --log-group-name <log-group-name> --kms-key-id <kms-key-arn> --region <region>`: 기존 로그 그룹에 KMS 키 연결.
        -   `aws logs create-log-group --log-group-name <new-log-group-name> --kms-key-id <kms-key-arn> --region <region>`: 새 로그 그룹 생성 시 KMS 키와 바로 연결.
- **KMS 키 정책 필요**: CloudWatch Logs 서비스(`logs.<region>.amazonaws.com`)가 해당 KMS 키를 사용할 수 있도록 키 정책에 권한(예: `kms:Encrypt`, `kms:Decrypt`, `kms:GenerateDataKey*` 등)을 명시적으로 추가해야 합니다.

## 10. CodeBuild 보안

### 10.1. VPC 내부 실행
- CodeBuild는 기본적으로 VPC 외부에서 실행되지만, VPC 내부 리소스(예: RDS 데이터베이스, 내부 ELB)에 접근해야 하는 경우 VPC 내부에서 실행하도록 구성할 수 있습니다.
- 설정: CodeBuild 프로젝트 -> 추가 구성 -> VPC, 서브넷, 보안 그룹 선택.

### 10.2. 환경 변수 암호화 (민감 정보 관리)
- CodeBuild 환경 변수에 암호 등 민감 정보를 평문으로 저장하는 것은 보안상 매우 위험합니다.
- **권장 방법**:
    1.  **SSM Parameter Store 참조**:
        -   환경 변수 유형을 "Parameter"로 선택.
        -   값으로 SSM Parameter Store에 저장된 파라미터의 이름(경로 포함, 예: `/CodeBuild/DBPassword`)을 지정.
        -   CodeBuild 실행 역할에 해당 SSM 파라미터 접근 권한 및 (SecureString의 경우) KMS 복호화 권한 필요.
    2.  **AWS Secrets Manager 참조**:
        -   환경 변수 유형을 "Secrets Manager"로 선택.
        -   값으로 Secrets Manager에 저장된 비밀의 이름 또는 ARN을 지정.
        -   CodeBuild 실행 역할에 해당 Secrets Manager 비밀 접근 권한 및 KMS 복호화 권한 필요.
- 런타임에 CodeBuild는 지정된 파라미터/비밀 값을 자동으로 가져와 컨테이너 환경 변수로 주입합니다.

## 11. AWS Nitro Enclaves

EC2 인스턴스 내에 격리된 컴퓨팅 환경(엔클레이브)을 생성하여 매우 민감한 데이터(PII, 건강 정보, 금융 정보 등)를 안전하게 처리할 수 있도록 하는 기술입니다.

- **주요 특징**:
    -   **완전 격리**: 엔클레이브는 호스트 EC2 인스턴스 및 다른 엔클레이브로부터 완전히 격리됩니다. 자체 커널, CPU, 메모리를 가집니다.
    -   **제한된 접근**: 영구 스토리지 없음, 대화형 액세스(SSH 등) 불가, 외부 네트워킹 없음.
    -   **보안 채널 통신**: 호스트 EC2 인스턴스와는 안전한 로컬 채널(vsock)을 통해서만 통신.
    -   **암호화 증명 (Attestation)**: 엔클레이브에서 실행되는 코드가 인증되었음을 증명하여, 신뢰할 수 있는 코드만 민감 데이터에 접근하도록 보장. KMS와 통합하여 엔클레이브만 특정 데이터에 접근할 수 있도록 설정 가능.
- **사용 사례**: 프라이빗 키 처리, 신용카드 결제 처리, 멀티파티 컴퓨팅 보호 등.
- **작동 방식**:
    1.  호환되는 Nitro 기반 EC2 인스턴스 실행 시 `EnclaveOptions`를 `true`로 설정.
    2.  Nitro CLI를 사용하여 애플리케이션을 엔클레이브 이미지 파일(EIF)로 변환.
    3.  Nitro CLI를 통해 EIF를 사용하여 EC2 인스턴스에 엔클레이브를 생성 및 실행.

---
