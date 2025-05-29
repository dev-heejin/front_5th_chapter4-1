## CloudFront 적용 전후 성능 비교 보고서
### 개요
- **목적**: 정적 리소스 요청 성능 개선을 위해 CDN 구성 적용
- **측정 조건**:
    - 브라우저 캐시 미사용(`Disable cache` 옵션 활성화)
    - Chrome DevTools > Network > Timing 탭에서 측정
    - 대상 리소스: js, svg, jpg 파일
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
* 엣지 서버가 더 가까워 네트워크 홉이 줄어듦의 의미란?
  * 요청과 응답이 짧은 거리로 빠르게 왕복된다는 구조적 이점을 의미
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
* TLS 핸드셰이크의 생략이란?
  * CloudFront가 이전 요청에서 이미 TLS 연결을 유지하고 있거나 브라우저가 같은 엣지 서버로 재요청하면서 기존 연결을 재사용하여 TLS 핸드셰이크 단계를 생략하거나 짧게 끝낼 수 있었다는 뜻
<br><br><br>

3. .jpg 파일
   - **테스트 대상 이미지**: `test_image.jpg` (크기: 8.8MB / 8,810,031 bytes)
   -  🔗 요청 URL
       - S3: `http://frontend-5th.s3-website-ap-southeast-2.amazonaws.com/test_image.jpg`
       - CloudFront: `https://d11xaxgq148rly.cloudfront.net/test_image.jpg`
<br>
   
| 항목 | CloudFront 미적용 (S3 직접 요청) | CloudFront 적용 |
|------|----------------------------------|-----------------|
| 총 응답 시간 (Total) | 2.67 s | **1.27 s** |
| TTFB (Waiting for server response) | 211.56 ms | **15.33 ms** |
| DNS Lookup | 0.49 ms | 6.66 ms |
| Content Download | 2.29 s | **1.25 s** |

✅ 주요 개선 포인트

| 항목 | 개선내용 |
|------|----------|
| 전체 응답 시간 52.4% 감소 | 2.67초 → 1.27초 |
| TTFB 92.7% 감소 | 서버 응답 대기 시간이 크게 단축됨 |
| 캐시 서버 응답 | CloudFront에서 `X-Cache: Hit from CloudFront`로 빠르게 응답 |
| 전송 시간 단축 | Edge 캐싱 덕분에 원본 S3보다 다운로드 속도 개선됨 |



### 결론 및 의의
- CloudFront 도입은 정적 리소스 응답 시간, 네트워크 연결 시간, 데이터 다운로드 효율성을 전체적으로 향상시킴
- 사용자 위치에 따라 CDN 엣지 서버가 자동 선택되어 전 세계 사용자 경험 품질 향상 가능
- 또한 이미지 리소스에도 성능 개선 효과가 확실히 있음
- 특히 첫 바이트 수신까지의 시간(TTFB)과 **총 응답 시간**이 현저히 감소
  <br><br><br>