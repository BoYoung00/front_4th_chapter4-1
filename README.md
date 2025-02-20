# 프론트엔드 배포 파이프라인

## 개요
<img src="https://github.com/user-attachments/assets/9a97cdb3-d035-424b-ae7e-4e4ea15a2ed9" width="700" />



GitHub Actions에 워크플로우를 작성해 다음과 같이 배포가 진행되도록 합니다.

 (사전작업: Ubuntu 최신 버전 설치)

1. Checkout 액션을 사용해 코드 내려받기
2. `npm ci` 명령어로 프로젝트 의존성 설치
3. `npm run build` 명령어로 Next.js 프로젝트 빌드
4. AWS 자격 증명 구성
5. 빌드된 파일을 S3 버킷에 동기화
6. CloudFront 캐시 무효화

## 주요 링크

- S3 버킷 웹사이트 엔드포인트: http://next-mock.s3-website-ap-southeast-2.amazonaws.com
- CloudFrount 배포 도메인 이름: [d19w432qnfaf3w.cloudfront.net](https://d19w432qnfaf3w.cloudfront.net)

## 주요 개념

#### - GitHub Actions과 CI/CD 도구:
GitHub Actions는 CI/CD(Continuous Integration / Continuous Deployment) 자동화 도구로, 코드 변경이 발생하면 빌드, 테스트, 배포 등의 작업을 자동으로 실행할 수 있도록 도와준다.
CI(Continuous Integration, 지속적 통합)는 개발자가 코드를 푸시할 때마다 자동으로 빌드 및 테스트하는 과정이고,
CD(Continuous Deployment, 지속적 배포)는 테스트를 통과한 코드를 자동으로 서버나 클라우드 환경에 배포하는 과정이다.

#### - S3와 스토리지:
S3(Simple Storage Service)는 AWS에서 제공하는 객체 스토리지 서비스로, 정적 웹사이트 호스팅, 백업, 데이터 저장 등의 용도로 사용된다.
Next.js에서 next export를 사용하면 정적 HTML 파일이 생성되는데, 이를 S3에 업로드하면 정적 사이트를 배포할 수 있다.

#### - CloudFront와 CDN:
CloudFront는 AWS에서 제공하는 CDN(Content Delivery Network) 서비스로, S3 등의 원본 서버에서 데이터를 가져와 전 세계 엣지 로케이션(Edge Location)에 캐시하여 빠르게 제공한다.
정적 웹사이트를 S3 + CloudFront 조합으로 배포하면, 사용자들은 가장 가까운 엣지 서버에서 데이터를 받아 빠르게 로딩할 수 있다.

#### - 캐시 무효화(Cache Invalidation):
CloudFront는 성능을 위해 파일을 캐시하지만, 새로운 배포 이후에도 기존 캐시된 파일이 유지될 수 있다.
이를 해결하려면 CloudFront Invalidation을 실행해 캐시를 삭제해야 한다.
예를 들어, aws cloudfront create-invalidation --paths "/*" 명령을 실행하면 CloudFront에 저장된 모든 파일을 새로고침하여 최신 버전으로 제공할 수 있다.

#### - Repository secret과 환경변수:
GitHub Actions에서 AWS 키, API 토큰 같은 민감한 정보를 보호하기 위해 사용하는 보안 변수이다.
Settings → Secrets and variables → Actions에서 설정하며, .env 파일 대신 GitHub Secrets에 저장하여 보안성을 높인다.
예를 들어, AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY 같은 값들은 코드에서 직접 노출되지 않고 ${{ secrets.AWS_ACCESS_KEY_ID }} 같은 형태로 사용된다.

## CDN과 성능최적화

## 1. 개요
본 보고서는 AWS CloudFront CDN 도입 전후의 **TTFB(Time To First Byte)** 개선 효과를 분석하는 것을 목표로 합니다. **TTFB**는 사용자가 요청한 페이지의 첫 번째 바이트가 서버에서 전달되는 데 걸리는 시간을 나타내며, 웹사이트의 반응 속도를 측정하는 중요한 지표입니다.

---

## 2. 테스트 환경
- **테스트 도구**: Chrome DevTools
- **측정 방법**: 동일한 네트워크 환경에서 5회 테스트 후 평균값을 산출
- **테스트 지역**: 서울 (한국)

---

## 3. TTFB 개선 결과

### 3.1 주요 TTFB 비교

| 지역  | CDN 미적용 TTFB (ms) | CDN 적용 후 TTFB (ms) | 개선율 |
|-------|----------------------|-----------------------|-------|
| 서울  | 164.23ms             | 14.9ms                 | **75% 감소** |

---

## 4. 분석 및 결론

### 4.1 주요 개선 사항
- **TTFB(Time To First Byte) 개선**: AWS CloudFront는 **Edge Location**을 활용하여, 사용자와 물리적으로 가까운 서버에서 빠르게 응답을 제공하여 TTFB를 크게 단축시켰습니다. CDN 도입 후, TTFB가 75% 개선되었습니다.
  
### 4.2 결론
AWS CloudFront CDN을 적용한 결과, **TTFB**가 75% 단축되었습니다. TTFB 개선은 전체 페이지 로딩 속도 향상에 큰 영향을 미치므로, CDN을 도입하는 것이 웹사이트 성능 최적화에 중요한 요소임을 알 수 있습니다.
