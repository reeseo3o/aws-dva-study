## IAM (Identity and Access Management)

### IAM 정책의 4가지 주요 요소
AWS에서 IAM 정책을 구성할 때 4가지 핵심 요소를 이해해야 합니다.

1. **Effect (효과)**
   - 정책이 Allow(허용)인지 Deny(거부)인지 정의합니다.

2. **Principal (주체)**
   - 정책을 적용할 사용자, 그룹, 역할을 지정합니다.

3. **Action (액션)**
   - 허용하거나 거부할 AWS 서비스 및 API 작업을 정의합니다.
   - 예: s3:PutObject, ec2:StartInstances 등

4. **Resource (리소스)**
   - 정책이 적용될 AWS 리소스를 지정합니다.
   - 예: 특정 S3 버킷, 특정 EC2 인스턴스 등

### 예제 IAM 정책 (S3 버킷에 대한 읽기 및 쓰기 권한 부여)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:user/Alice"
      },
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}
```