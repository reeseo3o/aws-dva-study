
# AWS CodeCommit

## 1. AWS CodeCommit 개요

AWS CodeCommit은 AWS에서 제공하는 완전 관리형 소스 코드 버전 관리 서비스입니다. GitHub나 GitLab과 유사하지만, AWS 인프라 내에서 운영되는 프라이빗 Git 저장소 서비스입니다.

### 1.1 주요 특징

| 기능 | 설명 |
|------|------|
| Git 호환성 | - 표준 Git 명령어 사용 가능 (clone, push, pull 등)<br>- 기존 Git 도구와 완벽한 호환 |
| 완전 관리형 | - 인프라 관리 불필요<br>- AWS가 서버 유지보수 담당<br>- 자동 확장성 제공 |
| 보안 | - 저장소 자동 암호화 (AWS KMS)<br>- 전송 중 데이터 암호화<br>- IAM을 통한 접근 제어 |
| 고가용성 | - 여러 AWS 가용 영역에 데이터 복제<br>- 자동 백업 및 복구 |
| 무제한 용량 | - 저장소 크기 제한 없음<br>- 파일 수 제한 없음 |
| AWS 서비스 통합 | - CodeBuild, CodeDeploy, CodePipeline과 통합<br>- CloudWatch, CloudTrail과 연동 |

### 1.2 사용 시나리오

1. **기업 내부 코드 관리**
   - 보안 정책상 외부 Git 호스팅 사용 불가 시
   - 민감한 코드의 내부 관리 필요 시

2. **CI/CD 파이프라인 구축**
   - AWS 네이티브 CI/CD 파이프라인 구성
   - CodePipeline과 연동한 자동화된 빌드/배포

3. **협업 개발**
   - 팀 단위 코드 리뷰
   - 브랜치 기반 개발
   - 풀 리퀘스트 워크플로우

## 2. CodeCommit 실무 활용

### 2.1 기본 설정 및 사용

```bash
# 1. IAM 사용자 생성 및 CodeCommit 권한 부여
# 2. Git 자격 증명 생성
aws iam create-service-specific-credential \
    --user-name YOUR_IAM_USER \
    --service-name codecommit.amazonaws.com

# 3. 저장소 생성
aws codecommit create-repository \
    --repository-name my-repo \
    --repository-description "My first CodeCommit repo"

# 4. 로컬에서 클론
git clone https://git-codecommit.region.amazonaws.com/v1/repos/my-repo
```

### 2.2 CI/CD 파이프라인 예시

1. **개발자 작업**
   ```bash
   git add .
   git commit -m "Update feature"
   git push origin main
   ```

2. **자동화된 파이프라인**
   - CodeCommit 푸시 감지
   - CodeBuild로 빌드 실행
   - CodeDeploy로 배포 수행

## 3. 모범 사례

### 3.1 보안

- IAM 역할과 정책을 사용한 최소 권한 원칙 적용
- 브랜치 보호 규칙 설정
- 커밋 서명 요구사항 설정
- 정기적인 접근 권한 검토

### 3.2 협업

- 브랜치 네이밍 규칙 수립
- 코드 리뷰 프로세스 정립
- 풀 리퀘스트 템플릿 사용
- 커밋 메시지 규칙 정의

### 3.3 모니터링

- CloudWatch 알람 설정
- CloudTrail을 통한 API 활동 추적
- 이벤트 규칙을 통한 알림 구성

## 4. DVA-C02 시험 주요 포인트

### 4.1 자주 출제되는 주제

1. **인증 및 권한**
   - IAM 사용자/역할 설정
   - Git 자격 증명 vs HTTPS 자격 증명
   - 브랜치 수준의 권한 제어

2. **통합 시나리오**
   - CodePipeline과의 연동
   - Lambda 트리거 구성
   - CloudWatch 이벤트 규칙

3. **보안 관련**
   - 암호화 설정
   - 교차 계정 접근
   - 보안 모범 사례

### 4.2 시험 준비 팁

- CodeCommit의 기본 개념과 특징 숙지
- IAM 정책과의 관계 이해
- 다른 AWS 개발자 도구와의 통합 시나리오 학습
- 실제 hands-on 실습 권장

## 5. 참고 사항

- CodeCommit 사용 자체는 무료이나, 저장소 크기와 API 호출에 따라 비용 발생
- Git LFS(Large File Storage) 지원
- 웹 콘솔에서 코드 브라우징 및 편집 가능
- 최대 파일 크기 제한: 2GB
- 브랜치 수 제한 없음 