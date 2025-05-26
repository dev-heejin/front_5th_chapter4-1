# front_5th_chapter4-1

## AWS S3 + CloudFront + GitHub Actions 배포 과정 정리
- Next.js 프로젝트(`out/` 폴더)를 정적 사이트로 빌드하고,
- AWS S3 + CloudFront를 통해 정적 사이트를 배포
- GitHub Actions를 사용해 CI/CD 자동화 구현

### 배포 흐름
1. `main` 브랜치에 푸시 또는 수동 실행 시 워크플로 시작
2. 코드 체크아웃 후 의존성 설치 (`npm ci`)
3. `npm run build`로 정적 파일(`out/`) 생성
4. AWS S3에 `aws s3 sync`로 업로드
5. CloudFront 배포의 캐시 무효화 수행

### ⚠️ 배포 과정에서 마주한 문제와 해결 방법
`Invalid bucket name` – S3 업로드 실패

- **상황**: `aws s3 sync` 명령 실행 시 버킷 이름 형식 오류
- **원인**:
    - Secrets에 등록된 값에 `"arn:aws:s3:::"` 혹은 `s3://` 접두어 포함
    - 또는 마스킹 문자열(`***`)을 그대로 등록함
- **해결 방법**:
    - **순수한 버킷 이름만 등록** (예: `my-app-bucket`)
    - ARN 형식은 CLI 명령어에서는 지원되지 않음



### 주요 링크

- S3 버킷 웹사이트 엔드포인트: http://frontend-5th.s3-website-ap-southeast-2.amazonaws.com/
- CloudFrount 배포 도메인 이름: https://d11xaxgq148rly.cloudfront.net/

### 주요 개념

- GitHub Actions과 CI/CD 도구: _________
- S3와 스토리지: _________
- CloudFront와 CDN: _________
- 캐시 무효화(Cache Invalidation): _________
- Repository secret과 환경변수: _________