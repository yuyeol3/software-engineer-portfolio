# 최유렬 | Software Engineer

[![GitHub](https://img.shields.io/badge/GitHub-yuyeol3-181717?logo=github)](https://github.com/yuyeol3)
[![Java](https://img.shields.io/badge/Java%20%7C%20Spring%20Boot-Backend-6DB33F?logo=springboot&logoColor=white)](#기술-스택)
[![TypeScript](https://img.shields.io/badge/TypeScript%20%7C%20NestJS-Full--stack-3178C6?logo=typescript&logoColor=white)](#기술-스택)

CJ올리브네트웍스 Software Engineer 지원 포트폴리오

---

## 직무 연관 역량

| CJ Software Engineer 업무·우대 역량 | 프로젝트에서 확인할 수 있는 경험 |
| --- | --- |
| 대내외 IT 사업 S/W 설계 및 개발 | 세무 판정 시스템, Kubernetes 정책 거버넌스 플랫폼, 금융상품 추천 서비스의 요구사항을 데이터 모델과 API로 구현 |
| WEB 서비스 개발 및 운영 | Spring Boot·NestJS API, React·Next.js 연동, 인증·인가, 배포 자동화, 장애 복구 흐름 경험 |
| Java·Spring Boot | 룰 엔진, OAuth2/JWT 인증, 약관 버전 관리, 금융상품 검색·수집 배치 개발 |
| AWS·DBMS | EC2·ECR·RDS 배포, PostgreSQL 트랜잭션·잠금, Flyway·Prisma 스키마 관리 |
| AI 활용 개발 | AI가 초안을 만들고 코드가 검증하는 구조, 상호명 분류 실험, 개발·리뷰 자동화 도구 구축 |

---

## Selected Projects

| 프로젝트 | 역할 | 핵심 기술 | 한 줄 성과 |
| --- | --- | --- | --- |
| [룰카드](#1-룰카드--결정론적-필요경비-판정-시스템) | Backend | Java 21, Spring Boot, PostgreSQL, AWS | 비결정적 AI 판정을 검증 가능한 규칙 엔진으로 분리하고 팀 개발·배포 흐름까지 자동화 |
| [Kyverno Governance Platform](#2-kyverno-governance-platform) | Backend | NestJS, Prisma, Kubernetes, Kyverno | 정책 예외의 승인과 실제 적용을 분리하고 재시도·만료·감사까지 상태로 관리 |
| [Y-FIN](#3-y-fin--청년-맞춤-금융상품-추천) | Backend / Data Pipeline | Java, Spring Boot, Spring Batch, PostgreSQL | 인증부터 개인화 추천, 외부 금융 데이터 수집까지 서비스 백엔드 전반 구현 |

---

## 1. 룰카드 — 결정론적 필요경비 판정 시스템

> 카카오테크 캠퍼스 4기 팀 프로젝트 · 6인 · **진행 중**  
> [팀 저장소](https://github.com/kakaotechcampus-4/ktc4-pusan-4) · [아키텍처](https://github.com/kakaotechcampus-4/ktc4-pusan-4/blob/develop/docs/architecture.md)

IT 프리랜서의 카드 거래가 필요경비에 해당하는지 법적 근거와 함께 설명하는 서비스입니다. 같은 거래에 답이 바뀌면 안 되는 세무 도메인의 특성을 고려해, **AI는 규칙 후보와 근거 초안을 만들고 실제 판정은 버전 관리되는 룰 엔진이 담당**하도록 설계했습니다.

### 담당한 문제와 해결

### 1. 규칙이 늘어날수록 충돌하는 판정 구조

- 차단형 규칙과 속성 누적형 규칙을 6개 관문으로 구분하고, 우선순위·구체성·ID에 따른 승자 결정 규칙을 코드로 고정했습니다.
- 잘못된 필드 타입, 근거 없는 확정 판정, 중복 속성 선언은 실행 중 장애가 아니라 **로딩 단계의 실패**로 바꿨습니다.
- 사용자 답변이 충돌하면 배치 전체를 중단하지 않고 해당 거래만 `확인 필요`로 안전하게 강등했습니다.
- 근거: [룰 엔진 구현 PR #10](https://github.com/kakaotechcampus-4/ktc4-pusan-4/pull/10), [구조·영속화 개선 PR #9](https://github.com/kakaotechcampus-4/ktc4-pusan-4/pull/9)

### 2. 판정 이력과 버전이 경쟁 상태에서 꼬일 수 있는 문제

- PostgreSQL 잠금을 이용해 사용자별 버전·개정 번호 발급을 직렬화했습니다.
- 룰과 법령 버전을 판정 결과에 함께 저장해, 나중에도 “어떤 규칙과 근거로 나온 결과인지” 재현할 수 있게 했습니다.
- 근거: [판정 DB·영속화 PR #11](https://github.com/kakaotechcampus-4/ktc4-pusan-4/pull/11)

### 3. 프론트엔드가 백엔드 완성 전까지 기다려야 하는 병목

- 문서화된 API 계약을 실행 가능한 NestJS 목 서버로 옮겼습니다.
- 업로드 멱등성, 판정 실행, 되묻기, 수동 오버라이드까지 실제 사용자 흐름을 구현하고 테스트해 프론트엔드가 병렬로 개발할 수 있게 했습니다.
- 근거: [API 계약 기반 목 서버 PR #41](https://github.com/kakaotechcampus-4/ktc4-pusan-4/pull/41)

### 4. 협업과 배포에서 반복되는 수작업

- 정적 분석·테스트 결과를 PR에 요약하는 코드 품질 봇을 추가했습니다.
- PR Discord 알림과 리뷰 리마인드를 자동화하고, fork PR에서는 비밀값을 읽지 못하는 GitHub Actions 보안 경계를 반영했습니다.
- GitHub Actions에서 이미지를 빌드해 ECR에 올리고 EC2가 완성된 이미지만 실행하는 단일 서버 배포 흐름을 구성했습니다.
- 근거: [코드 품질 봇 PR #37](https://github.com/kakaotechcampus-4/ktc4-pusan-4/pull/37), [리뷰 알림 PR #48](https://github.com/kakaotechcampus-4/ktc4-pusan-4/pull/48), [AWS 배포 PR #58](https://github.com/kakaotechcampus-4/ktc4-pusan-4/pull/58)

### 추가 검증: 상호명 분류 실험

키워드 규칙만으로 분류하기 어려운 카드 상호명을 보완할 수 있는지 별도 실험했습니다. 184만 개 고유 상호명을 char n-gram 선형 분류기로 학습하고, 공개 상가 데이터에 존재하지 않는 온라인·공과금 카테고리를 모델 대상에서 분리했습니다. 실제 카드 샘플에서는 규칙+모델 조합이 정확도 **0.60 → 0.82**, 커버리지 **63% → 100%**로 개선됐지만, 도메인 시프트와 미지원 카테고리의 한계도 함께 기록했습니다.

- [실험 저장소](https://github.com/yuyeol3/merchant-category-classifier)
- [규칙 대비 하이브리드 평가 보고서](https://github.com/yuyeol3/merchant-category-classifier/blob/main/REPORT.md)

---

## 2. Kyverno Governance Platform

> 부산대학교 졸업과제 · 3인 · Backend 담당  
> [저장소](https://github.com/pnucse-capstone2026/capstone-2026-team-30) · [시연 영상](https://www.youtube.com/watch?v=Gngvrf_XGBg)

![Kyverno Governance Platform 관리자 화면](https://raw.githubusercontent.com/pnucse-capstone2026/capstone-2026-team-30/main/docs/images/admin-dashboard.jpg)

Kubernetes 정책 위반을 조회하고, 한시적 예외의 요청·승인·적용·만료·감사 과정을 하나의 흐름으로 관리하는 플랫폼입니다. 승인 버튼의 성공과 실제 원격 클러스터 반영 성공을 동일하게 취급하지 않고, 외부 시스템 실패를 전제로 상태를 설계했습니다.

### 개인 기여

- JWT Access/Refresh Token 인증과 세션 회전, 로그아웃을 구현하고 동시 요청에서도 세션 상태가 일관되도록 트랜잭션을 보강했습니다.
- `ADMIN`, `APPROVER`, `REQUESTER`, `VIEWER` RBAC와 사용자별 클러스터 접근 범위를 적용했습니다.
- 정책 위반·예외 처리 API, 감사 로그, Kubernetes `PolicyException` 연동을 구현했습니다.
- 승인과 적용을 분리한 상태 모델을 만들고, Kubernetes API 일시 실패 시 지수 백오프 재시도와 조정 루프가 최종 상태를 복구하도록 했습니다.
- 대표 근거: [인증 구현](https://github.com/pnucse-capstone2026/capstone-2026-team-30/commit/3878df473a4936d06b0aec4c1ae7226f05e1347c), [RBAC 구현](https://github.com/pnucse-capstone2026/capstone-2026-team-30/commit/cbfe42f1e46fc222f1c2ed7e357fe879347e77dd), [세션 동시성 개선](https://github.com/pnucse-capstone2026/capstone-2026-team-30/commit/f355ff132fdfc03c49dbc9eb3764fedb3f44eda3), [예외 조정 개선](https://github.com/pnucse-capstone2026/capstone-2026-team-30/commit/95dd69ed2138529689e6dd0ac107d4adcea5582d)

### 팀 검증 결과

| 검증 항목 | 결과 |
| --- | ---: |
| 백엔드 자동화 테스트 | 58개 파일, 414개 케이스 통과 |
| PostgreSQL 통합 테스트 | 토큰 회전·세션 만료·예외 상태 전이 42개 케이스 |
| 동일 요청 20건 동시 승인 | 1건만 승인, 19건 차단, 중복 CR 0건 |
| 원격 EKS 예외 승인→적용 | p50 225.11ms, p95 429.03ms |
| EKS 위반 조회 50 VU | 96.79 RPS, 오류율 0% |

측정 수치와 환경, 현재 한계는 [프로젝트 README의 테스트·성능 평가](https://github.com/pnucse-capstone2026/capstone-2026-team-30#45-테스트-및-성능-평가)에 공개했습니다.

---

## 3. Y-FIN — 청년 맞춤 금융상품 추천

> APPTIVE 팀 프로젝트 · Backend 및 금융 데이터 수집 파이프라인  
> [서비스 백엔드](https://github.com/ApptiveDev/Fin-BE) · [데이터 수집기](https://github.com/ApptiveDev/Fin-API)

사용자의 소득·나이·가구·거래 조건을 바탕으로 예·적금 상품을 필터링하고 적합도와 예상 수익을 제공하는 서비스입니다. 서비스 API와 외부 데이터 수집기를 분리해 운영했습니다.

### 서비스 백엔드

- Google·Kakao OAuth2 로그인, JWT Access/Refresh Token, Cookie 기반 토큰 회전과 Spring Security 예외 처리를 구현했습니다.
- Refresh Token 갱신 경쟁 상태를 수정하고 재현 테스트로 동작을 고정했습니다.
- 약관을 단순 Boolean 값이 아니라 버전과 효력 시점을 가진 도메인으로 모델링하고, 필수 최신 약관 재동의 흐름을 구현했습니다.
- 사용자 응답에 따라 다음 입력 항목과 기본값이 달라지는 동적 폼, 맞춤 금융상품 필터링·정렬 로직을 구현했습니다.
- 근거: [인증 기반 구현 PR #3](https://github.com/ApptiveDev/Fin-BE/pull/3), [토큰 동시성 수정 PR #9](https://github.com/ApptiveDev/Fin-BE/pull/9), [약관 버전 관리 PR #12](https://github.com/ApptiveDev/Fin-BE/pull/12), [동적 입력 폼 PR #15](https://github.com/ApptiveDev/Fin-BE/pull/15)

### 금융 데이터 수집기

- 금융감독원과 온통청년 데이터를 Spring Batch로 수집하고 서로 다른 원본 구조를 공통 상품 모델로 정규화했습니다.
- 키워드 분류기를 책임별 컴포넌트로 나누고 점수 기반으로 개선했습니다.
- 수집원에서 사라진 상품을 비활성화하는 동기화 단계와 Testcontainers 기반 PostgreSQL 통합 테스트를 추가했습니다.
- 근거: [상품 동기화 PR #1](https://github.com/ApptiveDev/Fin-API/pull/1), [분류 개선 PR #3](https://github.com/ApptiveDev/Fin-API/pull/3)

---

## 기술 스택

| 구분 | 경험 |
| --- | --- |
| Backend | Java, Spring Boot, Spring Security, Spring Batch, NestJS, REST API, WebSocket |
| Data | PostgreSQL, MySQL, Redis, Prisma, JPA, Flyway, 트랜잭션·잠금 |
| Test | JUnit, Jest, Supertest, Testcontainers, API 계약·통합·동시성 테스트 |
| Infra | AWS EC2·ECR·RDS·EKS, Docker Compose, GitHub Actions, Kubernetes, Kyverno |
| Frontend | TypeScript, React, Next.js, Vite |
| AI / Data | LLM 구조화 출력·검증, RAG 설계, scikit-learn, char n-gram 분류 |

---

## More

- [GitHub 전체 프로젝트](https://github.com/yuyeol3?tab=repositories)
- [개발 블로그](https://yuyeol3.github.io/)
- [코드 구조를 빠르게 파악하기 위한 Code Map 도구](https://github.com/yuyeol3/maintain-code-map)
