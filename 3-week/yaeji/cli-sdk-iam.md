
## EC2 인스턴스 메타데이터 서비스 (IMDS)

EC2 인스턴스는 자신에 대한 정보를 IMDS(인스턴스 메타데이터 서비스)를 통해 얻을 수 있습니다. 이는 IAM 역할 없이도 인스턴스가 자신의 정보에 접근할 수 있는 강력한 기능입니다.

### 기본 접근 방법
- 메타데이터 접근 URL: `http://169.254.169.254/latest/meta-data`
- 조회 가능한 정보: 인스턴스 이름, 공개 IP, 프라이빗 IP, IAM 역할 이름, 자격증명 정보 등
- 유저데이터(EC2 인스턴스 실행 스크립트)와는 구분됩니다

### IMDS 버전
1. **IMDSv1**
   - 직접 URL에 접근하여 사용
   - 보안이 덜 강화됨

2. **IMDSv2** (Amazon Linux 2023부터 기본 활성화)
   - 보안 강화 버전
   - 두 단계 접근 방식:
     1. PUT 명령어로 세션 토큰 획득
     2. 획득한 토큰을 헤더에 포함시켜 IMDS URL 호출

## CLI/SDK와 다요소 인증(MFA)

### MFA를 통한 임시 세션 생성
1. STS GetSession Token API 사용
   - MFA 장치의 시리얼 번호와 토큰 코드 제공
   - 자격증명 기간 지정 가능
   - 결과로 임시 액세스 키 ID, 비밀 액세스 키, 세션 토큰 발급

### MFA 사용 예시
```bash
# MFA 토큰으로 임시 세션 토큰 발급
aws sts get-session-token --serial-number arn:aws:iam::XXXXXXXXXXXX:mfa/username --token-code 123456

# 발급된 자격증명으로 새 프로파일 설정
aws configure --profile mfa

# 자격증명 파일에 세션 토큰 추가 (~/.aws/credentials)
# [mfa]
# aws_access_key_id=XXXXXXXXXXXX
# aws_secret_access_key=XXXXXXXXXXXX
# aws_session_token=XXXXXXXXXXXX

# 프로파일 사용
aws s3 ls --profile mfa
```

## AWS 제한 및 지수 백오프 전략

### AWS 제한 유형
1. **API 비율 제한**
   - 연속 API 호출 횟수 제한 (예: EC2 DescribeInstances - 초당 100회)
   - 제한 초과 시 조절 오류(ThrottlingException) 발생

2. **서비스 제한(할당량)**
   - 사용 가능한 리소스 수 제한 (예: 온디맨드 표준 인스턴스 최대 1,152 vCPU)
   - 서비스 할당량 API로 증가 요청 가능

### 지수 백오프 전략
- **사용 시점**: 조절 오류(ThrottlingException) 또는 5xx 서버 오류 발생 시
- **작동 원리**: 재시도 간격을 지수적으로 증가 (1초 → 2초 → 4초 → 8초 → 16초...)
- **목적**: 서버 부하 감소 및 응답 가능성 증가
- **SDK 지원**: AWS SDK에 자동 구현되어 있음
- **주의**: 4xx 클라이언트 오류는 재시도하지 않음 (재시도해도 동일 오류 발생)

## 자격증명 우선순위

### CLI 자격증명 우선순위
1. 명령줄 옵션 (--region, --profile 등)
2. 환경 변수 (AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY 등)
3. CLI 자격증명 파일 (~/.aws/credentials)
4. CLI 구성 파일 (~/.aws/config)
5. 컨테이너 자격증명 (ECS 작업용)
6. 인스턴스 프로파일 자격증명 (EC2용)

### SDK 자격증명 우선순위 (Java 예시)
1. Java 시스템 속성
2. 환경 변수
3. 기본 자격증명 프로파일 파일
4. Amazon ECS 컨테이너 자격증명
5. EC2 인스턴스 프로파일 자격증명

### 주의사항
- 환경 변수가 EC2 인스턴스 프로파일보다 우선함
- 예시: EC2 인스턴스에 S3 접근 제한 역할을 부여해도, 환경 변수에 S3FullAccess 권한이 있는 자격증명이 설정되어 있다면 제한이 적용되지 않음
- 해결책: 환경 변수 설정 해제

## 자격증명 모범 사례
- 절대 자격증명을 코드에 하드코딩하지 않기
- AWS 서비스에서는 최대한 IAM 역할 사용하기
  - EC2 인스턴스 → EC2 인스턴스 역할
  - ECS 작업 → ECS 역할
  - Lambda 함수 → Lambda 역할
- AWS 외부에서는 환경 변수나 명명된 프로파일 사용하기

## AWS API 요청 서명 (SigV4)

### 요청 서명의 필요성
- AWS API 호출 시 요청자 인증을 위해 서명 필요
- AWS 자격증명(액세스 키, 시크릿 키)으로 서명
- SDK와 CLI는 자동으로 서명 처리

### 서명 전달 방법 (SigV4)
1. **HTTP 헤더 방식**
   - Authorization 헤더에 서명 포함
   - CLI 기본 방식

2. **쿼리 스트링 방식**
   - URL에 직접 서명 포함
   - 키 이름: X-Amz-Signature
   - 예시: `https://s3.amazonaws.com/bucket/key?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=...&X-Amz-Date=...&X-Amz-Expires=...&X-Amz-Signature=...`

### 서명 포함 요소
- 알고리즘 (AWS4-HMAC-SHA256)
- 자격증명 정보
- 날짜 및 유효 기간
- 실제 서명 값

## EC2 인스턴스에서 다중 환경 AWS CLI 관리

### 다중 환경 CLI 사용 시나리오
- EC2 인스턴스에서 개발/프로덕션 환경을 오가며 AWS CLI 사용
- 기본 IAM 역할과 액세스 키 세트 존재
- 추가 프로덕션 액세스 키 및 역할 정보 제공됨

### 최적의 해결방안
- AWS CLI 구성 파일에서 역할별 프로필 생성
- CLI 명령 실행 시 `--profile` 옵션으로 역할 전환

### 구성 예시
```ini
# ~/.aws/credentials
[dev]
aws_access_key_id = ABC...
aws_secret_access_key = XYZ...

[prod]
aws_access_key_id = DEF...
aws_secret_access_key = UVW...
```

```ini
# ~/.aws/config
[profile dev]
region = us-west-2

[profile prod]
region = us-east-1
role_arn = arn:aws:iam::123456789012:role/ProductionAccessRole
source_profile = dev
```

### CLI 사용 예시
```bash
# 프로덕션 프로필로 S3 버킷 조회
aws s3 ls --profile prod
```

### 다른 방법과 비교
| 방법 | 평가 | 이유 |
|------|------|------|
| CLI 프로필 사용 | ✅ 권장 | 안전하고 관리가 용이함 |
| 인스턴스 사용자 데이터에 저장 | ❌ 비권장 | 보안에 취약하고 시작 시에만 실행 |
| 인스턴스 메타데이터에 저장 | ❌ 비권장 | 읽기 전용이며 자격증명 저장에 부적합 |
| 인스턴스 프로필 생성 | ❌ 비권장 | EC2-역할 연결용으로 CLI 프로필과 다름 |

### 보안 모범 사례
1. 자격증명 파일 권한 설정 (600)
2. 정기적인 액세스 키 로테이션
3. 최소 권한 원칙 준수
4. 환경별 명확한 프로필 이름 사용

### 주의사항
- 환경 변수가 프로필 설정보다 우선순위가 높음
- 프로덕션 자격증명은 특별히 안전하게 관리
- MFA가 필요한 경우 세션 토큰 활용

#### 환경 변수로 인한 권한 우회 사례 및 주의점

- 환경 변수(`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` 등)에 강한 권한의 자격증명이 설정되어 있으면,  
  EC2 인스턴스에 S3 접근 제한 역할을 부여해도 **환경 변수의 자격증명이 우선 적용**되어 제한이 무시됩니다.
- 예시:  
  EC2 인스턴스에 S3 읽기 전용 역할만 부여했지만,  
  환경 변수에 S3FullAccess 권한이 있는 액세스 키가 설정되어 있으면  
  해당 인스턴스에서 S3 전체 접근이 가능해짐 → **의도치 않은 권한 상승 및 보안 사고 위험**
- **해결책:**  
  - 환경 변수에 불필요한 자격증명이 남아 있지 않은지 항상 점검  
  - `unset AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY AWS_SESSION_TOKEN` 등으로 환경 변수 해제  
  - 시스템 재시작 시 자동으로 환경 변수가 설정되지 않도록 쉘 프로파일(`~/.bash_profile`, `~/.zshrc` 등)도 점검
- **실무 팁:**  
  - IAM 역할 기반 인증이 가능한 환경(EC2, Lambda 등)에서는 환경 변수 방식 대신 역할 기반 인증을 우선 사용  
  - 환경 변수 사용이 불가피하다면, 최소 권한 원칙을 반드시 적용하고,  
    사용 후 즉시 환경 변수를 해제하여 권한 오남용을 방지
