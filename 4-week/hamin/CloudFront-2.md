## CloudFront 서명된 URL/쿠키

서명된 URL과 쿠키는 CloudFront 콘텐츠에 대한 접근을 제한하고 인증된 사용자만 콘텐츠에 접근할 수 있도록 하는 기능입니다.

### 서명된 URL vs 서명된 쿠키

- **서명된 URL**:
  - 개별 파일에 대한 접근 제어에 적합
  - 쿠키를 지원하지 않는 클라이언트에 유용
  - 특정 파일에 대한 접근만 제한할 때 사용

- **서명된 쿠키**:
  - 여러 제한된 파일에 대한 접근이 필요할 때 유용
  - 현재 URL 구조 변경 없이 접근 제어 가능
  - 비디오 스트리밍처럼 여러 파일에 접근해야 할 때 편리

### 주요 구성 요소

1. **만료 시간**: URL/쿠키가 유효한 기간 지정
2. **IP 주소/범위**: 특정 IP에서만 접근 가능하도록 제한 (선택 사항)
3. **신뢰할 수 있는 서명자**: CloudFront 키 페어를 사용해 URL/쿠키 서명
4. **정책 설명**: 제한 사항을 정의하는 JSON 정책 (URL 파라미터 또는 Base64로 인코딩)

### 구현 방법

1. **키 페어 생성**: AWS 계정의 루트 사용자를 통해 CloudFront 키 페어 생성
2. **애플리케이션에 비공개 키 저장**: 서명 생성에 사용
3. **서명 생성**: AWS SDK 또는 서명 생성 유틸리티 사용
4. **URL 또는 쿠키에 서명 추가**: 생성된 서명을 URL 파라미터 또는 쿠키 값으로 추가

### 사용 사례

- **유료 콘텐츠**: 결제한 사용자만 접근 가능한 프리미엄 콘텐츠
- **회원 전용 콘텐츠**: 로그인한 사용자만 접근 가능한 리소스
- **시간 제한 액세스**: 특정 기간 동안만 유효한 미디어 파일 접근 권한
- **콘텐츠 보호**: 무단 배포를 방지하기 위한 디지털 콘텐츠 보호

<br>

## CloudFront 서명된 URL - 키 그룹 + 실습

키 그룹은 CloudFront 서명된 URL 및 쿠키를 관리하는 더 안전하고 유연한 방법을 제공합니다. 기존의 신뢰할 수 있는 서명자 기능을 대체하는 권장 방식입니다.

### 키 그룹의 장점

- **IAM 권한 분리**: 루트 계정 액세스 없이 키 관리 가능
- **키 순환**: 쉬운 퍼블릭 키 교체 및 여러 키 동시 사용 가능
- **재사용**: 여러 배포에서 동일한 키 그룹 사용 가능
- **더 나은 보안**: 더 안전한 키 관리 방식 제공

### 키 그룹 설정 단계

1. **퍼블릭/프라이빗 키 페어 생성**:
   ```bash
   # OpenSSL을 사용하여 키 페어 생성
   openssl genrsa -out private_key.pem 2048
   openssl rsa -pubout -in private_key.pem -out public_key.pem
   ```

2. **CloudFront 콘솔에서 퍼블릭 키 추가**:
   - CloudFront 콘솔 → 보안 → 퍼블릭 키
   - 퍼블릭 키 파일 내용 붙여넣기
   - 키 이름 지정

3. **키 그룹 생성**:
   - CloudFront 콘솔 → 보안 → 키 그룹
   - 새 키 그룹 생성 및 퍼블릭 키 추가

4. **CloudFront 배포에 키 그룹 연결**:
   - 배포 설정 → 동작 설정 → 제한된 뷰어 액세스 활성화
   - 서명된 URL 또는 서명된 쿠키 선택
   - 생성한 키 그룹 선택

### 서명된 URL 생성 예제 (Node.js)

```javascript
const crypto = require('crypto');
const fs = require('fs');

function createSignedUrl(resourceUrl, privateKeyPath, keyPairId, expiresIn) {
  // 만료 시간 계산 (현재 시간 + expiresIn(초))
  const expireTime = Math.floor(Date.now() / 1000) + expiresIn;
  
  // 정책 생성
  const policy = {
    Statement: [
      {
        Resource: resourceUrl,
        Condition: {
          DateLessThan: {
            'AWS:EpochTime': expireTime
          }
        }
      }
    ]
  };
  
  // 정책을 JSON 문자열로 변환하고 Base64로 인코딩
  const policyString = JSON.stringify(policy);
  const policyBase64 = Buffer.from(policyString).toString('base64');
  
  // 개인 키 로드
  const privateKey = fs.readFileSync(privateKeyPath, 'utf8');
  
  // 서명 생성
  const signer = crypto.createSign('RSA-SHA1');
  signer.update(policyBase64);
  const signature = signer.sign(privateKey, 'base64');
  
  // URL 파라미터 구성
  const urlParams = new URLSearchParams({
    'Expires': expireTime,
    'Key-Pair-Id': keyPairId,
    'Signature': signature.replace(/\+/g, '-').replace(/=/g, '_').replace(/\//g, '~')
  });
  
  // 서명된 URL 반환
  return `${resourceUrl}?${urlParams.toString()}`;
}

// 사용 예
const signedUrl = createSignedUrl(
  'https://d1234abcd.cloudfront.net/protected-video.mp4',
  './private_key.pem',
  'K2JCJMDEHXQW5F',
  3600 // 1시간
);

console.log(signedUrl);
```

### 실습 단계

1. **환경 준비**:
   - CloudFront 배포 설정 (S3 버킷을 오리진으로)
   - 프라이빗/퍼블릭 키 페어 생성
   - 키 그룹 생성 및 배포에 연결

2. **제한된 콘텐츠 업로드**:
   - S3 버킷에 테스트 파일 업로드
   - CloudFront 배포 동작 설정에서 해당 경로에 대한 제한 설정

3. **서명된 URL 생성**:
   - 위 코드 예제를 사용하여 서명된 URL 생성
   - 필요에 따라 IP 주소 제한, 시간 제한 등 추가

4. **테스트**:
   - 일반 URL로 접근 시 "Access Denied" 오류 확인
   - 서명된 URL로 접근 시 콘텐츠 접근 확인
   - 만료 시간 이후 접근 불가 확인

### 주요 고려사항

- **비공개 키 보안**: 비공개 키는 안전하게 보관하고 권한이 있는 서버에서만 접근 가능하도록 관리
- **URL 길이 제한**: 서명된 URL이 너무 길어질 수 있으므로 짧은 정책 사용 고려
- **시계 동기화**: 서명 생성 서버의 시간이 정확해야 함
- **HTTPS 사용**: 서명된 URL을 안전하게 전송하기 위해 HTTPS 사용 권장

<br>

## 고급 개념

### 가격 등급 (Price Class)

CloudFront의 엣지 로케이션은 전 세계에 걸쳐 분포되어 있으며, 지역에 따라 데이터 전송 비용이 다릅니다. AWS는 비용 최적화를 위해 여러 가격 등급을 제공합니다.

#### 가격 등급의 종류

- **Price Class All (모든 엣지 로케이션 사용)**
  - 전 세계 모든 CloudFront 엣지 로케이션에 콘텐츠 배포
  - 최고의 성능과 가장 낮은 지연 시간 제공
  - 가장 비싼 가격 등급
  - 글로벌 서비스에 적합

- **Price Class 200 (대부분의 엣지 로케이션 사용)**
  - 북미, 유럽, 아시아, 중동, 아프리카의 대부분 지역 포함
  - 가장 비싼 지역(남미, 호주 등)을 제외
  - 성능과 비용 사이의 균형점
  - 대부분의 글로벌 서비스에 적합

- **Price Class 100 (가장 저렴한 지역만 사용)**
  - 북미와 유럽의 엣지 로케이션만 포함
  - 가장 저렴한 가격 등급
  - 주로 북미와 유럽의 사용자를 대상으로 하는 서비스에 적합

#### 가격 등급 선택 시 고려사항

- **대상 사용자의 지리적 위치**: 주요 사용자층이 있는 지역을 포함하는 가격 등급 선택
- **성능 요구사항**: 지연 시간에 민감한 애플리케이션의 경우 더 많은 엣지 로케이션 필요
- **예산 제약**: 비용 최적화가 중요한 경우 더 제한된 가격 등급 고려
- **트래픽 패턴**: 트래픽이 특정 지역에 집중된 경우 해당 지역을 포함하는 최소한의 가격 등급 선택

<br>

### 다중 오리진 & 오리진 그룹

CloudFront에서는 콘텐츠 유형이나 경로에 따라 다른 오리진으로 요청을 라우팅할 수 있으며, 고가용성을 위한 오리진 그룹 설정도 가능합니다.

#### 다중 오리진 (Multiple Origins)

- **다양한 오리진 유형 지원**:
  - Amazon S3 버킷
  - Application Load Balancer
  - EC2 인스턴스
  - MediaPackage 채널
  - 사용자 지정 오리진(자체 웹서버)

- **경로 패턴 기반 라우팅**:
  - `/images/*` → S3 버킷으로 라우팅
  - `/api/*` → ALB로 라우팅
  - `/videos/*` → MediaStore로 라우팅

- **콘텐츠 유형별 최적화**:
  - 정적 콘텐츠는 S3에 저장
  - 동적 콘텐츠는 EC2/ALB에서 처리
  - 미디어 콘텐츠는 MediaPackage에서 처리

#### 오리진 그룹 (Origin Groups)

오리진 그룹은 장애 조치(failover) 메커니즘을 제공하여 고가용성을 보장합니다.

- **구성 요소**:
  - 주 오리진(Primary Origin): 기본적으로 요청을 처리하는 오리진
  - 부 오리진(Secondary Origin): 주 오리진에 장애 발생 시 요청을 처리하는 대체 오리진

- **장애 조치 메커니즘**:
  1. CloudFront가 주 오리진에 요청 전송
  2. 주 오리진이 지정된 HTTP 오류 코드(4xx, 5xx)를 반환하면 실패로 간주
  3. CloudFront가 동일한 요청을 부 오리진으로 자동 재전송
  4. 부 오리진의 응답이 사용자에게 전달

- **사용 사례**:
  - 중요한 웹 애플리케이션의 고가용성 보장
  - 리전 간 장애 조치 구현
  - 서로 다른 AWS 서비스 간의 백업 메커니즘 구축

<br>

### 필드 수준 암호화 (Field-Level Encryption)

필드 수준 암호화는 민감한 사용자 데이터를 엣지에서 암호화하여 전체 애플리케이션 스택을 통해 보호하는 기능입니다.

#### 기본 개념

- **추가적인 보안 계층**: HTTPS로 제공되는 전송 중 암호화 외에 추가 보호 제공
- **특정 필드만 암호화**: 전체 요청이 아닌 신용카드 번호, 사회보장번호 등 민감한 필드만 선택적 암호화
- **비대칭 암호화 사용**: 공개 키로 암호화하고 개인 키로만 복호화 가능

#### 작동 원리

1. **필드 지정**: CloudFront 배포에서 암호화할 최대 10개의 데이터 필드 지정
2. **엣지에서 암호화**: 사용자의 요청이 엣지 로케이션에 도달하면 지정된 필드를 공개 키로 암호화
3. **암호화된 상태로 전송**: 암호화된 데이터는 로드 밸런서, 애플리케이션 서버 등 전체 인프라를 통과
4. **선택적 복호화**: 개인 키를 보유한 인증된 애플리케이션 컴포넌트만 데이터 복호화 가능

#### 구성 단계

1. **키 페어 생성**: RSA 키 페어 생성 (퍼블릭/프라이빗 키)
2. **공개 키 업로드**: CloudFront 콘솔에 공개 키 업로드
3. **암호화 프로필 생성**: 어떤 필드를 암호화할지 정의하는 프로필 설정
4. **캐시 동작에 구성 연결**: 특정 URL 패턴에 대한 암호화 설정 적용

#### 사용 사례

- **결제 정보 보호**: 신용카드 번호, CVV 등의 결제 정보 보호
- **개인 식별 정보 보안**: 사회보장번호, 의료 정보 등의 민감한 개인 정보 보호
- **규제 준수**: PCI DSS, HIPAA 등의 규제 요구사항 준수 지원
- **데이터 유출 위험 감소**: 백엔드 시스템 침해 시에도 암호화된 데이터는 보호됨

<br>

### 실시간 로그 (Real-Time Logs)

CloudFront 실시간 로그는 배포에 대한 요청 정보를 실시간으로 전송하여 콘텐츠 전송 성능을 모니터링하고 분석할 수 있게 해줍니다.

#### 주요 특징

- **지연 시간 최소화**: 요청이 처리된 후 몇 초 내에 로그 레코드 수신
- **샘플링 비율 설정**: 모든 요청 또는 특정 비율의 요청만 로깅 가능
- **필드 선택**: 필요한 로그 필드만 선택하여 비용 최적화
- **Kinesis Data Streams 통합**: 로그가 Kinesis Data Streams로 직접 전송됨

#### 구성 요소

- **CloudFront 배포**: 로깅을 활성화할 배포
- **Kinesis Data Streams**: 로그 데이터를 수신하는 스트림
- **처리 시스템**: Kinesis 스트림의 데이터를 처리하는 시스템
  - Lambda 함수: 실시간 처리 및 알림
  - Firehose: 장기 저장 및 분석용 데이터 전송

#### 구성 단계

1. **Kinesis Data Stream 생성**: CloudFront 로그를 수신할 스트림 설정
2. **IAM 권한 설정**: CloudFront가 Kinesis에 쓸 수 있는 권한 부여
3. **실시간 로그 구성 생성**: CloudFront 콘솔에서 로그 설정
   - 샘플링 비율, 필드, 엔드포인트 지정
4. **로그 처리 파이프라인 구축**:
   - Lambda 함수를 트리거하여 실시간 분석
   - Firehose를 통해 S3, Redshift, Elasticsearch로 전송

#### 사용 사례

- **실시간 모니터링**: 콘텐츠 전송 성능 및 가용성 모니터링
- **보안 분석**: 의심스러운 요청 패턴 감지 및 대응
- **사용자 경험 개선**: 오류 발생 시 즉시 알림 및 대응
- **트래픽 분석**: 실시간 트래픽 패턴 분석 및 용량 계획

#### 아키텍처 예시

**기본 실시간 처리**:
1. 사용자가 CloudFront에 요청
2. CloudFront가 요청 처리 및 실시간 로그 생성
3. 로그가 Kinesis Data Streams로 전송
4. Lambda 함수가 로그 레코드 처리
5. 필요한 경우 알림 전송 또는 데이터베이스 업데이트

**장기 저장 및 분석**:
1. CloudFront 로그가 Kinesis Data Streams로 전송
2. Kinesis Data Firehose가 데이터 스트림에서 데이터 소비
3. Firehose가 데이터를 S3, Redshift, Elasticsearch 등으로 전송
4. Amazon Athena 또는 QuickSight로 장기 분석 수행

