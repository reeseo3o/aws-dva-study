# AWS Security Token Service (STS)

AWS Security Token Service(STS)는 AWS 리소스에 대한 액세스를 제어하기 위해 임시 보안 자격 증명을 생성하고 제공하는 웹 서비스입니다. 이러한 임시 자격 증명은 최대 1시간(AssumeRole의 경우 최소 15분에서 최대 12시간까지 설정 가능, GetSessionToken의 경우 최소 15분에서 최대 36시간까지 설정 가능)동안 유효합니다.

## 주요 STS API

STS는 다양한 API를 제공하며, 시험에 자주 출제되는 주요 API는 다음과 같습니다.

*   **`AssumeRole`**: 가장 핵심적인 API로, 동일 계정 또는 교차 계정 내에서 IAM 역할을 수임(assume)하여 임시 자격 증명을 얻습니다.
*   **`AssumeRoleWithSAML`**: SAML(Security Assertion Markup Language) 2.0 기반 인증을 통해 로그인한 사용자에게 임시 보안 자격 증명을 부여합니다.
*   **`AssumeRoleWithWebIdentity`**: 웹 자격 증명 공급자(예: Facebook, Google, Amazon Cognito)를 통해 로그인한 사용자에게 역할을 수임하도록 합니다. (현재는 주로 Amazon Cognito 자격 증명 풀 사용 권장)
*   **`GetSessionToken`**: MFA(Multi-Factor Authentication)가 활성화된 IAM 사용자 또는 AWS 루트 계정 사용자를 위해 임시 자격 증명을 반환합니다.
*   **`GetFederationToken`**: 연동된 사용자(federated user)를 위한 임시 자격 증명을 반환합니다. 일반적으로 IAM 사용자를 생성하는 대신 외부 ID 시스템을 사용할 때 사용됩니다.
*   **`GetCallerIdentity`**: API 호출에 사용된 IAM 사용자 또는 역할의 세부 정보(계정 ID, 사용자 ARN, 역할 ARN 등)를 반환합니다. 현재 사용 중인 자격 증명을 확인할 때 유용합니다.
*   **`DecodeAuthorizationMessage`**: AWS API 호출이 거부되었을 때 반환되는 인코딩된 오류 메시지를 디코딩하여 실패 원인에 대한 자세한 정보를 제공합니다.

**시험 주요 출제 API**: `AssumeRole`, `GetSessionToken`, `GetCallerIdentity`, `DecodeAuthorizationMessage`

## DecodeAuthorizationMessage API

AWS STS DecodeAuthorizationMessage API는 AWS 요청에 대한 응답으로 반환된 인코딩된 메시지에서 요청의 권한 부여 상태에 대한 추가 정보를 디코딩합니다.

### 작동 원리

1. **권한 부여 실패 시나리오**:
   * 사용자가 요청한 작업을 수행할 권한이 없는 경우, 해당 요청은 `Client.UnauthorizedOperation` 응답(HTTP 403 응답)을 반환합니다.
   * 일부 AWS 작업은 이러한 권한 부여 실패에 대한 세부 정보를 제공하는 인코딩된 메시지를 추가로 반환합니다.

2. **메시지 인코딩의 이유**:
   * 권한 부여 상태 세부 정보가 작업을 요청한 사용자에게 공개되어서는 안 되는 특권 정보로 구성될 수 있기 때문입니다.
   * 보안상의 이유로 메시지가 인코딩되어 있습니다.

3. **필요한 IAM 권한**:
   * 권한 부여 상태 메시지를 디코딩하려면 IAM 정책을 통해 사용자에게 `DecodeAuthorizationMessage` (`sts:DecodeAuthorizationMessage`) 작업을 요청할 수 있는 권한이 부여되어야 합니다.

### IAM 정책 예시

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowDecodeAuthorizationMessage",
            "Effect": "Allow",
            "Action": "sts:DecodeAuthorizationMessage",
            "Resource": "*"
        }
    ]
}
```

### AWS CLI 사용 예시

```bash
aws sts decode-authorization-message --encoded-message [인코딩된_메시지]
```

### 디코딩된 메시지 예시

```json
{
    "DecodedMessage": {
        "allowed": false,
        "explicitDeny": false,
        "matchedStatements": [],
        "failures": [
            "no-matching-statement"
        ],
        "context": {
            "principal": {
                "id": "AROAXXXXXXXXXXXXXXXXX",
                "arn": "arn:aws:iam::123456789012:role/example-role"
            },
            "action": "s3:PutObject",
            "resource": "arn:aws:s3:::example-bucket/*",
            "conditions": {}
        }
    }
}
```

### 주의사항

* AWS KMS나 외부 라이브러리로는 디코딩할 수 없습니다.
* IAM 서비스가 아닌 STS 서비스를 통해서만 디코딩이 가능합니다.
* 보안상의 이유로 디코딩 권한은 신중하게 부여해야 합니다.

### 활용 사례

* 권한 문제 디버깅
* IAM 정책 문제 해결
* 보안 감사
* 권한 설정 최적화

## `AssumeRole` 작동 원리

1.  **IAM 역할 정의**: 대상 계정(동일 계정 또는 다른 계정)에 IAM 역할을 정의합니다.
    *   **신뢰 정책(Trust Policy)**: 어떤 주체(principal)가 이 역할을 수임할 수 있는지 정의합니다.
    *   **권한 정책(Permissions Policy)**: 이 역할을 수임한 주체가 어떤 AWS 리소스와 작업에 액세스할 수 있는지 정의합니다.
2.  **`AssumeRole` API 호출**: 역할을 수임하려는 주체가 STS의 `AssumeRole` API를 호출하여 해당 IAM 역할의 임시 자격 증명을 요청합니다.
3.  **임시 자격 증명 반환**: STS는 요청이 유효하고 신뢰 정책 및 권한 정책에 부합하는지 확인한 후, 임시 보안 자격 증명(액세스 키 ID, 비밀 액세스 키, 세션 토큰)을 반환합니다.
4.  **자격 증명 유효 기간**: 기본적으로 1시간이며, 최소 15분에서 최대 12시간까지 설정 가능합니다.

### 교차 계정 액세스

다른 AWS 계정의 리소스에 액세스해야 할 경우, 대상 계정에 IAM 역할을 생성하고, 해당 역할을 수임할 수 있도록 현재 계정의 IAM 사용자 또는 역할에 신뢰 관계를 설정합니다. 그 후 `AssumeRole` API를 호출하여 대상 계정의 리소스에 접근합니다.

## STS와 MFA (`GetSessionToken`)

MFA(Multi-Factor Authentication)는 보안을 강화하는 중요한 수단입니다. `GetSessionToken` API는 MFA 인증을 거친 사용자에게 임시 자격 증명을 발급하는 데 사용됩니다.

*   **용도**: MFA 디바이스로 인증한 후, 일반적인 AWS API 호출(AWS Management Console 외부에서 이루어지는 호출, 예: CLI, SDK)에 사용할 수 있는 임시 자격 증명을 얻습니다.
*   **필수 조건**: IAM 정책에서 `aws:MultiFactorAuthPresent:true` 조건을 사용하여 MFA 인증을 강제할 수 있습니다.
    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "ec2:StopInstances",
                    "ec2:TerminateInstances"
                ],
                "Resource": "*",
                "Condition": {
                    "Bool": {
                        "aws:MultiFactorAuthPresent": "true"
                    }
                }
            }
        ]
    }
    ```
*   **반환값**: 액세스 키 ID, 비밀 액세스 키, **세션 토큰**, 그리고 자격 증명 만료 시간을 반환합니다. API 호출 시 이 세 가지 정보를 모두 사용해야 합니다.

# 고급 IAM 개념

## IAM 정책 평가 로직

IAM 정책 평가는 다음 순서로 이루어집니다.

1.  **명시적 거부(Explicit Deny)**: 정책에 명시적 거부가 있으면 최종 결정은 **거부**입니다.
2.  **명시적 허용(Explicit Allow)**: 명시적 거부가 없고, 명시적 허용이 있으면 최종 결정은 **허용**입니다.
3.  **기본값(Default Deny/Implicit Deny)**: 명시적 거부도 없고 명시적 허용도 없으면 최종 결정은 **거부**입니다.

**중요**: 명시적 거부는 항상 명시적 허용보다 우선합니다.

## IAM 정책과 S3 버킷 정책의 상호 작용

IAM 정책(사용자, 그룹, 역할에 연결)과 S3 버킷 정책(버킷에 연결)은 함께 평가되어 최종 권한을 결정합니다.

*   어느 한 쪽에서라도 명시적 허용이 있고, 다른 쪽에서 명시적 거부가 없다면 작업은 허용될 수 있습니다.
*   어느 한 쪽이라도 명시적 거부가 있다면 작업은 거부됩니다.
*   **평가**: (IAM 정책 OR S3 버킷 정책) 중 하나라도 허용하고, 양쪽 모두에 명시적 거부가 없어야 최종적으로 허용됩니다. 즉, 최종 권한은 두 정책의 **통합(union)**으로 평가됩니다.

**예시 시나리오**:

1.  **EC2 인스턴스 역할에 S3 읽기/쓰기 허용, S3 버킷 정책 없음**: EC2는 S3 버킷에 읽기/쓰기 **가능**.
2.  **EC2 인스턴스 역할에 S3 읽기/쓰기 허용, S3 버킷 정책에서 해당 역할 명시적 거부**: EC2는 S3 버킷에 읽기/쓰기 **불가능** (명시적 거부 우선).
3.  **EC2 인스턴스 역할에 S3 권한 없음 (빈 역할), S3 버킷 정책에서 해당 역할 명시적 읽기/쓰기 허용**: EC2는 S3 버킷에 읽기/쓰기 **가능**.
4.  **EC2 인스턴스 역할에 S3 명시적 거부, S3 버킷 정책에서 해당 역할 명시적 허용**: EC2는 S3 버킷에 읽기/쓰기 **불가능** (명시적 거부 우선).

## 동적 정책 (IAM Policy Variables)

여러 사용자에게 각자의 홈 디렉터리(예: S3 버킷 내 `/home/<username>/`)에만 접근 권한을 부여해야 할 때, 사용자마다 개별 정책을 생성하는 것은 비효율적입니다.

**해결책**: IAM 정책 변수 `${aws:username}`을 사용한 동적 정책을 활용합니다.

*   단일 IAM 정책을 생성하여 여러 사용자에게 적용할 수 있습니다.
*   정책이 평가될 때 `${aws:username}` 변수는 실제 API를 호출한 사용자의 이름으로 동적으로 대체됩니다.
*   **예시**:
    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "s3:GetObject",
                    "s3:PutObject",
                    "s3:DeleteObject"
                ],
                "Resource": [
                    "arn:aws:s3:::my-bucket/home/${aws:username}/*"
                ]
            },
            {
                 "Effect": "Allow",
                 "Action": "s3:ListBucket",
                 "Resource": "arn:aws:s3:::my-bucket",
                 "Condition": {"StringLike": {"s3:prefix": ["home/${aws:username}/*"]}}
            }
        ]
    }
    ```

## 인라인 정책 vs 관리형 정책

AWS IAM 정책에는 세 가지 주요 유형이 있습니다.

1.  **AWS 관리형 정책 (AWS Managed Policies)**:
    *   AWS가 생성하고 관리하는 정책입니다.
    *   일반적인 사용 사례(예: AdministratorAccess, PowerUserAccess, ReadOnlyAccess)를 위해 미리 정의되어 있습니다.
    *   AWS 서비스가 업데이트되거나 새로운 API가 추가되면 AWS가 자동으로 업데이트합니다.
    *   재사용 가능하며 여러 IAM 보안 주체(사용자, 그룹, 역할)에 연결할 수 있습니다.

2.  **고객 관리형 정책 (Customer Managed Policies)**:
    *   사용자가 직접 생성하고 관리하는 독립형 정책입니다.
    *   세분화된 권한 제어가 필요할 때 권장되는 모범 사례입니다.
    *   재사용 가능하며 여러 IAM 보안 주체에 연결할 수 있습니다.
    *   버전 관리가 가능하여 변경 사항을 추적하고 롤백할 수 있습니다.
    *   중앙에서 변경 관리가 용이합니다.

3.  **인라인 정책 (Inline Policies)**:
    *   단일 IAM 사용자, 그룹 또는 역할에 직접 포함(embed)되는 정책입니다.
    *   정책과 보안 주체 간에 엄격한 일대일 관계를 가집니다.
    *   보안 주체를 삭제하면 인라인 정책도 함께 삭제됩니다.
    *   버전 관리를 지원하지 않습니다.
    *   재사용이 어렵습니다.
    *   정책 크기에 제한이 있을 수 있습니다 (예: 사용자당 인라인 정책의 총 크기 제한).
    *   특별한 경우(정책이 특정 보안 주체에만 엄격하게 적용되어야 하는 경우)를 제외하고는 관리형 정책 사용이 권장됩니다.

## IAM `iam:PassRole` 권한

AWS 서비스가 다른 AWS 서비스를 대신하여 작업을 수행해야 할 때 IAM 역할을 해당 서비스에 전달해야 합니다. 이때 `iam:PassRole` 권한이 필요합니다.

*   **필요성**: EC2 인스턴스에 IAM 역할을 할당하여 S3에 접근하게 하거나, Lambda 함수에 역할을 할당하여 DynamoDB를 호출하게 하는 경우 등, 한 서비스가 다른 서비스의 권한을 위임받아 실행할 때 사용됩니다.
*   **작동 방식**:
    1.  사용자(또는 서비스)는 IAM 역할을 특정 AWS 서비스(예: EC2, Lambda)에 전달합니다.
    2.  이를 위해서는 역할을 전달하는 주체에게 `iam:PassRole` 권한이 있어야 하며, 전달하려는 특정 역할(또는 역할 패턴)에 대한 권한이어야 합니다.
    3.  전달받는 서비스는 해당 역할의 신뢰 정책(Trust Relationship)에 명시된 경우에만 역할을 수임할 수 있습니다.
*   **신뢰 정책 (Trust Policy/Trust Relationship)**: 역할이 어떤 서비스(또는 계정, 사용자 등)를 신뢰하고 해당 서비스가 역할을 수임하도록 허용할지를 정의합니다. 예를 들어, EC2 인스턴스 프로파일 역할의 신뢰 정책에는 `ec2.amazonaws.com` 서비스가 명시되어 있어야 합니다.
*   **일반적인 정책 구문**:
    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": "ec2:*", // 사용자의 EC2 관련 작업 권한
                "Resource": "*"
            },
            {
                "Effect": "Allow",
                "Action": "iam:PassRole",
                "Resource": "arn:aws:iam::ACCOUNT_ID:role/S3AccessRole", // 특정 역할(S3AccessRole)만 전달하도록 제한
                "Condition": {
                    "StringEquals": {"iam:PassedToService": "ec2.amazonaws.com"} // EC2 서비스에 전달될 때만 허용 (선택적)
                }
            }
        ]
    }
    ```
*   `iam:GetRole` 권한은 종종 `iam:PassRole`과 함께 부여되어 전달될 역할의 세부 정보를 확인할 수 있도록 합니다.

# Microsoft Active Directory (AD) 및 AWS Directory Service

## Microsoft Active Directory (AD) 개요

*   Windows 서버 환경에서 사용되는 디렉터리 서비스입니다.
*   **객체 데이터베이스**: 사용자 계정, 컴퓨터, 프린터, 파일 공유, 보안 그룹 등의 정보를 저장하고 관리합니다.
*   **중앙 집중식 보안 관리**: 온프레미스 환경에서 사용자 인증 및 권한 부여를 중앙에서 관리합니다.
*   **구조**: 객체는 트리(tree) 형태로 구성되며, 여러 트리가 모여 포리스트(forest)를 이룹니다.
*   **도메인 컨트롤러(DC)**: AD의 핵심 서버로, 사용자 인증 및 정책 적용을 담당합니다.

## AWS Directory Service

AWS 클라우드에서 Active Directory 기능을 사용하거나 온프레미스 AD와 통합할 수 있는 관리형 서비스입니다.

1.  **AWS Managed Microsoft AD**:
    *   AWS에서 직접 관리하는 실제 Microsoft Active Directory입니다.
    *   **온프레미스 AD와 신뢰 관계(Trust Relationship) 설정 가능**: 양방향 또는 단방향 신뢰를 통해 온프레미스 사용자와 AWS AD 사용자가 상호 리소스에 접근할 수 있습니다.
    *   MFA(Multi-Factor Authentication)를 지원합니다.
    *   사용자 관리는 AWS 내 AD 또는 온프레미스 AD 양쪽에서 이루어질 수 있습니다(신뢰 관계 구성에 따라 다름).
    *   **에디션**: Standard Edition (최대 3만 객체), Enterprise Edition (최대 50만 객체).

2.  **AD Connector**:
    *   온프레미스 Active Directory로 인증 요청을 **프록시(proxy) 또는 리디렉션**하는 디렉터리 게이트웨이입니다.
    *   AWS에 사용자 정보를 저장하지 않습니다. 모든 인증은 온프레미스 AD에서 처리됩니다.
    *   MFA 지원 (일반적으로 RADIUS 서버와 통합).
    *   **주요 용도**: 기존 온프레미스 AD 자격 증명을 사용하여 AWS 서비스(예: EC2 인스턴스 로그인, WorkSpaces, Amazon Chime)에 액세스하고자 할 때 사용합니다.
    *   **사용자 관리**: 온프레미스 AD에서만 이루어집니다.

3.  **Simple AD**:
    *   AWS에서 제공하는 독립적인 관리형 디렉터리입니다. Samba 4 기반으로 하며, Microsoft AD와 호환되는 기능을 제공합니다.
    *   **Microsoft AD가 아닙니다.**
    *   온프레미스 AD와 신뢰 관계를 설정할 수 **없습니다**.
    *   **주요 용도**: Linux 워크로드에 대한 기본적인 AD 기능이 필요하거나, 저렴한 비용으로 소규모 환경에서 디렉터리 서비스가 필요할 때 사용합니다.
    *   MFA를 기본적으로 지원하지 않을 수 있으며, 고급 AD 기능(예: 그룹 정책의 모든 기능, 포리스트 트러스트)이 제한될 수 있습니다.

### 활용 사례

*   Windows 기반 EC2 인스턴스를 AWS Directory Service의 도메인에 조인하여 중앙 집중식 사용자 관리 및 인증을 수행할 수 있습니다.
*   AWS Management Console, AWS 애플리케이션(WorkSpaces, QuickSight 등)에 AD 자격 증명으로 로그인할 수 있습니다.

### 시험 핵심 포인트

*   **온프레미스 사용자를 프록시**: AD Connector
*   **AWS Cloud에서 사용자 관리하며 MFA 지원, 온프레미스와 신뢰 관계 가능**: AWS Managed Microsoft AD
*   **온프레미스 요소가 없는 독립형 AD 호환 디렉터리**: Simple AD

(참고: Amazon Cognito 사용자 풀은 사용자 인증 및 권한 부여를 위한 서비스이지만, 전통적인 디렉터리 서비스와는 다른 범주로 분류됩니다.)

## 문제 - 온프레미스 LDAP과 AWS IAM 통합: SAML 미지원 환경에서의 최적 솔루션

### 문제 배경

- 온프레미스 데이터센터에서 LDAP(예: OpenLDAP, Microsoft AD 등) 기반의 인증 시스템을 사용 중이나, 이 LDAP이 SAML 2.0을 지원하지 않는 상황.
- AWS 리소스 접근을 위해 IAM과 연동이 필요함.

### 각 옵션별 설명 및 정답 근거

#### 1. **AWS와 LDAP 간의 액세스를 관리하기 위해 AWS IAM Identity Center 서비스를 구현**
- **오답**: AWS IAM Identity Center(구 AWS SSO)는 SAML, OIDC 등 표준 프로토콜을 지원하는 IdP와 연동할 수 있지만, SAML 미지원 LDAP과는 직접 연동 불가.
- **LDAP 연동**은 AD Connector 또는 AWS Managed Microsoft AD를 통해서만 가능하며, 이 역시 SAML 또는 Kerberos 기반 인증이 필요.
- **즉, SAML 미지원 LDAP은 직접 연동 불가**.

#### 2. **LDAP 자격 증명이 업데이트될 때마다 IAM 자격 증명을 순환하기 위해 IAM 역할을 생성**
- **오답**: IAM 자격 증명(Access Key, Secret Key)을 수동으로 관리/순환하는 것은 보안상 위험하며, 자동화/통합 인증 목적에 부합하지 않음.
- **LDAP과 IAM의 직접적 연동이 불가**하므로, 이 방식은 권장되지 않음.

#### 3. **온프레미스 데이터 센터에서 사용자 지정 ID 브로커 애플리케이션을 만들고 STS를 사용하여 단기 AWS 자격 증명을 발급**
- **정답**: SAML 미지원 LDAP 환경에서 AWS와 연동하려면, **사용자 지정 ID 브로커**(Custom Identity Broker) 애플리케이션을 구축해야 함.
- 이 브로커는 LDAP(또는 AD 등)에서 사용자를 인증한 뒤, AWS STS의 `AssumeRole` 또는 `GetFederationToken` API를 호출하여 **임시 보안 자격 증명**(Access Key, Secret Key, Session Token)을 발급받음.
- 브로커는 이 임시 자격 증명을 사내 애플리케이션 또는 사용자가 사용할 수 있도록 전달.
- **실무 예시**: AWS 공식 문서에서도 SAML 미지원 LDAP 환경에서는 Custom Broker + STS 조합을 권장함.
- **장점**: LDAP 사용자 인증과 AWS 리소스 접근을 안전하게 분리, 임시 자격 증명으로 보안성 강화, 자격 증명 자동 만료.

#### 4. **LDAP 식별자와 AWS 자격 증명을 참조하는 IAM 정책을 설정**
- **오답**: IAM 정책만으로 LDAP 사용자와 AWS 자격 증명을 직접 연결할 수 없음.
- 정책은 AWS 내의 주체(사용자, 역할, 그룹)에만 적용되며, 외부 LDAP 사용자와 직접 매핑 불가.

---

### 추가 설명: STS, AssumeRole, GetFederationToken, Custom Identity Broker

#### **AWS STS(Security Token Service)**
- 임시 보안 자격 증명을 발급하는 서비스.
- `AssumeRole`, `GetFederationToken` 등 다양한 API 제공.
- 임시 자격 증명은 만료 시간이 짧아 보안성이 높음.

#### **Custom Identity Broker의 동작 원리**
1. 사용자가 온프레미스 LDAP에 인증.
2. 브로커 애플리케이션이 인증된 사용자를 대신해 AWS STS의 `AssumeRole` 또는 `GetFederationToken` 호출.
3. STS가 임시 자격 증명(Access Key, Secret Key, Session Token) 반환.
4. 브로커가 이 자격 증명을 사용자 또는 내부 애플리케이션에 전달.
5. 사용자는 이 임시 자격 증명으로 AWS 리소스에 접근.

#### **AssumeRole vs GetFederationToken**
- **AssumeRole**: 주로 교차 계정, 역할 기반 접근 제어에 사용. 신뢰 정책(Trust Policy) 필요.
- **GetFederationToken**: 외부 인증 시스템(예: LDAP)과 연동 시, 임시로 AWS 리소스 접근 권한 부여에 사용.

#### **실무 적용 예시**
- 사내 포털에서 LDAP 인증 → 브로커가 STS 호출 → 임시 자격 증명으로 S3, DynamoDB 등 접근.
- 자격 증명 만료 시 자동 갱신, 보안성 강화.

#### **관련 AWS 서비스와의 차이점**
- **AWS Directory Service**: AD 기반 인증에 적합, LDAP만 사용하는 경우 제한적.
- **AD Connector**: 온프레미스 AD와 AWS 서비스 연동, LDAP만 사용하는 경우에는 직접적 연동 불가.
- **IAM Identity Center**: SAML/OIDC 기반 IdP 필요, SAML 미지원 LDAP은 직접 연동 불가.

---

### 결론

- **SAML 미지원 LDAP 환경에서 AWS IAM과 통합하려면, 온프레미스에 Custom Identity Broker를 구축하고, 이 브로커가 AWS STS를 통해 임시 자격 증명을 발급하는 방식이 가장 적합**.
- 이 방식은 AWS 공식 아키텍처 및 실무 사례에서도 권장됨.
