# front_5th_chapter4-1

## 주요 링크

- S3 버킷 웹사이트 엔드포인트: http://frontend-5th.s3-website-ap-southeast-2.amazonaws.com/
- CloudFrount 배포 도메인 이름: https://d11xaxgq148rly.cloudfront.net/

## AWS S3 + CloudFront + GitHub Actions 배포 과정 정리
- Next.js 프로젝트(`out/` 폴더)를 정적 사이트로 빌드하고,
- AWS S3 + CloudFront를 통해 정적 사이트를 배포
- GitHub Actions를 사용해 CI/CD 자동화 구현

### 배포 흐름
1. `main` 브랜치에 푸시 또는 수동 실행 시 워크플로 시작
2. 코드 체크아웃 후 의존성 설치 (`npm ci`)
3. `npm run build`로 정적 파일(`out/`) 생성
4. AWS S3에 `aws s3 sync`로 업로드
5. CloudFront 배포의 캐시 무효화 수행 (이후 새 요청은 S3 오리진으로 요청 => CloudFront 가 응답을 엣지에 새로 캐싱 후 최신 콘텐츠 제공)


### ⚠️ 배포 과정에서 마주한 문제와 해결 방법
1. `Invalid bucket name` – S3 업로드 실패

- **상황**: `aws s3 sync` 명령 실행 시 버킷 이름 형식 오류
- **원인**:
    - Secrets에 등록된 값에 `"arn:aws:s3:::"` 혹은 `s3://` 접두어 포함
    - 또는 마스킹 문자열(`***`)을 그대로 등록함
- **해결 방법**:
    - **순수한 버킷 이름만 등록** (예: `my-app-bucket`)
    - ARN 형식은 CLI 명령어에서는 지원되지 않음
      <br><br><br>
2. `Content-Encoding 누락` – CloudFront 배포 후 응답 헤더에서 압축 정보가 보이지 않음

- **상황**: CloudFront를 통해 배포한 정적 리소스의 응답에서 `Content-Encoding` 헤더가 보이지 않음
- **원인**:
    - 브라우저가 이미 캐시한 리소스를 재요청할 때, CloudFront 엣지 로케이션이 `304 Not Modified` 응답을 반환함
    - `304` 응답은 본문이 없기 때문에, `Content-Encoding`, `Content-Length` 등의 본문 관련 헤더는 **HTTP 표준에 따라 생략**됨
    - 압축은 되어 있어도 헤더가 생략되므로, DevTools에서 이를 확인할 수 없음
- **해결 방법**:
    - 브라우저의 개발자 도구(DevTools) → **Network 탭에서 "Disable cache"** 옵션을 활성화하고 새로고침
    - 강제로 **CloudFront가 200 OK 응답**을 반환하게 하여, 전체 응답 헤더(`Content-Encoding`, `Content-Length` 등)를 다시 확인할 수 있음
      <br><br><br>
3. `AccessDenied` – IAM 사용자로 로그인 시 비용 및 보안 관련 리소스 접근 실패
- **상황**: IAM 사용자로 AWS 콘솔에 로그인한 뒤 아래 API/서비스에 접근 시 모두 `AccessDenied` 오류 발생
- **원인**:
  - 해당 IAM 사용자에게 필요한 권한이 포함된 IAM 정책이 부여되지 않음
  - 일부 서비스(특히 비용 관련 서비스)는 기본적으로 IAM 사용자에게 권한이 없으며, 루트 사용자 또는 명시적으로 권한이 부여된 사용자만 접근 가능
  - `Billing Access`가 IAM 사용자에게 허용되지 않은 경우도 존재
- **해결 방법**:
  - 루트 사용자로 로그인하여 IAM 콘솔에서 **정책 추가 생성**
    ```json 
    "Action": [
             "ce:GetCostAndUsage",
             "ce:GetCostForecast",
             "cost-optimization-hub:ListEnrollmentStatuses",
             "securityhub:DescribeHub",
             "servicecatalog:ListApplications"
           ],

### 주요 개념
- GitHub Actions과 CI/CD 도구
  - **CI/CD 도구**
    - 소프트웨어 개발 프로세스를 자동화하여, 코드 변경 사항을 자동으로 빌드, 테스트, 배포하는 도구
    - **지속적 통합(Continuous Integration)**: 개발자가 코드를 자주 통합하여, 빌드와 테스트를 자동으로 수행하는 프로세스
    - **지속적 배포(Continuous Deployment)**: 코드 변경 사항을 자동으로 프로덕션 환경에 배포하는 프로세스
    - 수동 빌드/테스트는 실수와 누락의 가능성이 크고, 자동화된 CI/CD 도구를 사용하면 일관성과 신뢰성을 높일 수 있음
  - **GitHub Actions**
    - Gihub에 통합된 CI/CD(지속적 통합/배포) 자동화 도구
    - 코드를 푸시하거나, PR생성 시 미리 정의한 워크플로우에 따라 자동으로 빌드,테스트,배포 등의 작업을 수행할 수 있음
    - 주요 개념
      - Workflow: 자동화 작업 전체의 정의로, yaml 파일로 작성하며 `.github/workflows/` 디렉토리에 위치
      - Job: 워크플로우 내에서 실행되는 작업 단위, 여러 개의 Job이 병렬 또는 순차적으로 실행 가능
      - Step: Job 내에서 실행되는 개별 작업, 명령어 또는 스크립트
      - Action: 재사용 가능한 작업 단위 (예: checkout, upload-artifact 등)
      - Runner: 워크플로우를 실행하는 서버
  
---

- S3와 스토리지
  - **스토리지(Storage)**
    - 데이터를 저장하고 관리하는 기술 혹은 장치
    - 블록스토리지, 파일스토리지, 오브젝트스토리지
  - **S3 (Simple Storage Service)**
    - AWS의 오브젝트 스토리지 서비스: 인터넷을 통해 무제한의 파일을 저장하고 읽을 수 있는 저장소
    - 주요 개념
      - Bucket: S3의 최상위 저장 단위(디렉토리 역할)
      - Object: S3에 저장되는 파일(데이터) 단위
      - Key: S3에서 Object를 식별하는 고유한 이름

---

- CloudFront와 CDN: 
  - **CDN (Content Delivery Network)**
    - 전 세계 여러 지역에 분산된 서버(엣지 서버)를 통해 콘텐츠를 사용자에게 더 빠르고 안정적으로 제공하는 기술/네트워크
    - 구성 요소
      - Edge Location: CDN의 엣지 서버가 위치한 지역, 사용자와 가까운 곳에 위치하여 빠른 응답 제공
      - Origin: 콘텐츠의 원본 서버 (예: S3 버킷, EC2 인스턴스 등)
      - Cache: 엣지 서버에 저장된 콘텐츠로, 사용자 요청 시 빠르게 제공됨
    - 동작방식: 사용자가 웹페이지나 파일 요청 -> 가장 가까운 엣지 서버에서 콘텐츠 제공(캐시 되어있는 경우) -> 엣지서버에 없으면 원본 서버에서 콘텐츠를 가져와 캐싱 후 전달 -> 동일 요청 시 캐시에서 바로 응답
  - **CloudFront**
    - 전 세계 엣지 로케이션(Edge Location)을 통해 정적/동적 콘텐츠를 사용자에게 더 빠르고 안전하게 전달하는 CDN(Content Delivery Network) 서비스
    - 주요 개념
      - Origin: CloudFront가 콘텐츠를 가져오는 원본 서버 (예: S3 버킷)
      - Cache: CloudFront 엣지 로케이션에 저장된 콘텐츠로, 사용자 요청 시 빠르게 제공됨
    - S3 버킷을 오리진으로 지정하고 CloudFront를 사용하면 전 세계에 빠른 콘텐츠 제공 가능
    - 동작흐름: 사용자가 웹사이트에 요청 -> CloudFront 엣지 로케이션이 요청 확인 -> 캐시 HIT ? 엣지에서 응답 : 오리진(S3/EC2 등)에서 가져와 엣지에서 저장 후 응답

---

- 캐시 무효화(Cache Invalidation)
  - 이미 CDN(또는 브라우져, 프록시 등)에 저장된 캐시된 콘텐츠를 더 이상 유효하지 않도록 처리해, 새로운 버전의 콘텐츠를 강제로 불러오게 하는 과정
  - CDN이나 브라우저는 성능 향상을 위해 정적 리소스 (HTML, CSS, JS 등)를 로컬이나 엣지 서버에 캐시하는데, 콘텐츠를 업데이트한 후에도 이전 캐시가 남아 있으면 사용자는 오래된 파일을 보게 된다 - 이때 캐시 무효화를 통해 새로운 콘텐츠를 불러오도록 강제할 수 있다.
  - **CloudFront 캐시 무효화**
    - CloudFront에서 캐시된 콘텐츠를 무효화하려면, AWS Management Console, AWS CLI, SDK 등을 사용하여 캐시 무효화 요청을 생성
    - 무효화 요청은 특정 파일 또는 경로를 지정하여 해당 콘텐츠를 삭제하고, 다음 요청 시 오리진에서 새로운 콘텐츠를 가져오도록 함
    - 예: `aws cloudfront create-invalidation --distribution-id <배포 ID> --paths "/*"` (모든 파일 무효화)

---

- Repository secret과 환경변수
  - 외부에 노출되면 안되는 값이나 설정을 안전하게 관리하고, 코드에서 유연하게 사용할 수 있도록 도와주는 수단
  - **Repository secret**
    - GitHub 리포지토리에서 사용하는 비밀 값(예: API 키, 비밀번호 등)을 안전하게 저장하는 기능
    - 워크플로우 실행 시 환경변수로 주입되어 코드에서 사용할
    - 예: `env: AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}`
  - 환경변수 (Environment Variable)
  - 애플리케이션이나 스크립트 실행 시 필요한 설정 값이나 비밀 정보를 외부에서 주입하여 코드의 유연성과 이식성을 높이는 방식(동일 코드를 개발/테스트/운영 환경 어디서든 실행 가능하게 해줌)
  - 예: `NODE_ENV=production`, `PORT=3000`, `.env` 파일에서 불러오는 API_URL 등 


## CloudFront 적용 전후 성능 비교 보고서(캐시 미사용 기준)
### 개요
- **목적**: 정적 리소스 요청 성능 개선을 위해 CDN 구성 적용
- **측정 조건**:
  - 브라우저 캐시 미사용(`Disable cache` 옵션 활성화)
  - Chrome DevTools > Network > Timing 탭에서 측정
  - 대상 리소스: js, svg 파일
  <br><br>
### 측정 결과
1. js 파일

| 항목 | CloudFront 미적용 (S3 직접) | CloudFront 적용 |
|------|-----------------------------|-----------------|
| 총 응답 시간 (Total) | **398.21 ms** | **68.06 ms** |
| TTFB (Waiting for server response) | 226.09 ms | 16.83 ms |
| 초기 연결 시간 (Initial connection) | 147.56 ms | 31.44 ms |
| DNS Lookup | 12 µs | 16.84 ms |
| Content Download | 5.20 ms | 0.38 ms |

✅ 주요 개선 포인트

| 항목 | 개선 내용 |
|------|-----------|
| 전체 로딩 시간 약 83% 감소 | 사용자 체감 속도 향상 |
| TTFB 약 92.6% 감소 | 서버에서 첫 응답이 도달하는 시간 감소 (226ms → 17ms) |
| 초기 연결/SSL 수립 시간 개선 | 엣지 서버가 더 가까워 네트워크 홉이 줄어듦 |
| 콘텐츠 다운로드 시간 감소 | CDN 서버의 최적화된 네트워크와 압축 전송 |

<br><br><br>

2. svg 파일

| 항목 | CloudFront 미적용 (S3 직접 요청) | CloudFront 적용 |
|------|----------------------------------|-----------------|
| 총 응답 시간 (Total) | 249.85 ms | **31.29 ms** |
| TTFB (Waiting for server response) | 190.79 ms | **25.61 ms** |
| DNS Lookup | 0 µs | 1.32 ms |
| Content Download | 0.19 ms | 3.00 ms |

✅ 주요 개선 포인트

| 항목 | 개선내용                                           |
|------|------------------------------------------------|
| 전체 응답 시간 87.5% 감소 | 249.85ms → 31.29ms                             |
| TTFB 86.5% 감소 | 서버 응답 지연이 크게 개선됨                               |
| 연결 지연 제거 | CloudFront는 엣지 서버와 연결이 미리 유지되었거나 TLS 핸드셰이크가 생략됨 |
| Content Download 소폭 증가 | 캐시 서버가 압축 없이 전달했거나 물리적 거리 차이. 하지만 전체 시간엔 영향 미미 |
<br><br><br>
### 결론 및 의의
- CloudFront 도입은 정적 리소스 응답 시간, 네트워크 연결 시간, 데이터 다운로드 효율성을 전체적으로 향상시킴
- 사용자 위치에 따라 CDN 엣지 서버가 자동 선택되어 전 세계 사용자 경험 품질 향상 가능
- 또한 이미지 리소스(SVG)에도 성능 개선 효과가 확실히 있음
- 특히 첫 바이트 수신까지의 시간(TTFB)과 **총 응답 시간**이 현저히 감소
  <br><br><br>
