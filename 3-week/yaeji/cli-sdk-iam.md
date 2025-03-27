
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
