# AWS CLI, SDK, IAM

---

## EC2 인스턴스 메타데이터

EC2 인스턴스는 자체적으로 메타데이터를 가지고 있으며, 이는 인스턴스 내부에서 HTTP 요청을 통해 조회할 수 있습니다.

- 조회 경로: `http://169.254.169.254/latest/meta-data/`
- 주요 정보:
  - AMI ID
  - 인스턴스 ID
  - 호스트 이름
  - IAM 역할 이름
  - 보안 그룹 등
- `user-data`도 같은 방식으로 조회 가능 (`latest/user-data`)

**활용 예시:** 부팅 시 초기 설정 자동화, 스크립트 실행 등

---

## AWS CLI 프로필

AWS CLI는 여러 개의 프로필(Profile)을 지원하여 서로 다른 계정이나 권한을 쉽게 전환할 수 있습니다.

- 프로필 생성:
  ```bash
  aws configure --profile myprofile
  ```
- 프로필 사용:
  ```bash
  aws s3 ls --profile myprofile
  ```
- 프로필 정보는 `~/.aws/config`와 `~/.aws/credentials` 파일에 저장됩니다.

---

## MFA가 있는 AWS CLI 사용

MFA가 활성화된 IAM 사용자로 CLI를 사용하려면, 세션 토큰(STS)을 통해 임시 자격 증명을 생성해야 합니다.

1. MFA 토큰을 입력하여 세션 토큰 발급:
   ```bash
   aws sts get-session-token \
     --serial-number arn:aws:iam::123456789012:mfa/your-user \
     --token-code 123456
   ```
2. 반환된 `AccessKeyId`, `SecretAccessKey`, `SessionToken`을 환경 변수나 CLI 프로파일에 설정하여 사용

---

## AWS SDK 개요

AWS SDK는 Java, Python(Boto3), JavaScript 등 다양한 언어로 AWS 서비스에 접근할 수 있게 해주는 라이브러리입니다.

- 자격 증명 탐색 순서 (Credential Provider Chain):
  1. 환경 변수 (`AWS_ACCESS_KEY_ID` 등)
  2. 명시적으로 지정한 자격 증명
  3. CLI 프로파일 (`~/.aws/credentials`)
  4. EC2 인스턴스 메타데이터의 IAM 역할


  -> 언제 SDK를 쓰느냐?? 는 중요함
  -> 기본 리전을 지정하지 않으면 api 호출을 위해 sdk에서 us-east-1 기본으로 선택된다 -> 시험에 나올지도



---

## 지수 백오프 및 서비스 제한 증가

AWS API 호출 실패 시, 재시도 전략으로 지수 백오프(Exponential Backoff)를 사용할 수 있습니다.

- 기본 아이디어: 재시도 간 간격을 점점 늘림 (예: 1초 → 2초 → 4초)
- AWS SDK에서는 기본적으로 이 전략을 자동 적용함
- 특정 서비스 호출 제한(Quota)에 도달한 경우, Service Quota Increase 요청을 통해 상향 조정 가능


지수 백오프는 언제 사용해야하나? -> 조절 오류가 발생했을 때

조절 오류는 api 호출을 많이 했을 때 발생하며 일반적으로 이에 대한 해답은 지수 백오프입니다. (시험에 나올 수 있음)

즉 지수 백오프를 재시도 해야하는 경우는 쉽게 서버 에러로 이해하면 될 듯 (500에러)

---
