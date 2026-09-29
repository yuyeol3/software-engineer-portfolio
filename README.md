<!-- markdownlint-disable MD013 -->

# 최유렬 | Software Engineer Portfolio

[![GitHub](https://img.shields.io/badge/GitHub-yuyeol3-181717?logo=github)](https://github.com/yuyeol3)
[![Java](https://img.shields.io/badge/Java-Spring%20Boot-6DB33F?logo=springboot&logoColor=white)](#기술-스택)
[![TypeScript](https://img.shields.io/badge/TypeScript-NestJS-3178C6?logo=typescript&logoColor=white)](#기술-스택)

CJ올리브네트웍스 Software Engineer 지원 포트폴리오

동시성, 상태 전이, 외부 연동 실패처럼 재현하기 어려운 문제를 테스트와 측정으로 확인하는 백엔드 개발자입니다. Java·Spring 기반 서비스와 데이터 파이프라인을 주로 구현했고, LLM은 결과를 그대로 신뢰하기보다 출력 검증·실패 제어·평가 기준을 함께 설계해 사용합니다.

## 직무 연관 경험

| 공고의 업무·우대사항 | 관련 경험 |
| --- | --- |
| S/W 설계 및 개발 | 세무 판정, 금융상품 추천, Kubernetes 정책 관리 시스템의 데이터 모델·API 구현 |
| WEB 서비스 개발 및 운영 | 인증·인가, 트랜잭션, 외부 API 연동, 배포 및 실패 복구 흐름 구현 |
| Java·Spring Boot | 룰 엔진, OAuth2/JWT 인증, 약관 버전 관리, 개인화 검색, Spring Batch 수집기 |
| AWS·DBMS | EC2·ECR·RDS·EKS, PostgreSQL·MySQL, Flyway·Prisma, 잠금과 동시성 제어 |
| LLM·생성형 AI 연동 | Y-FIN의 Gemini 정규화 파이프라인, 룰카드의 AI·결정론적 코드 책임 분리 |
| AI 협업 역량 | 생성 코드의 취약한 테스트를 폐기하고 평가 하네스의 품질·토큰 비용을 조정한 기록 |

## 프로젝트 요약

| 프로젝트 | 역할 | 핵심 기여 | 대표 검증 |
| --- | --- | --- | --- |
| [Y-FIN](#1-y-fin--청년-맞춤-금융상품-추천) | Backend / Data Pipeline | 인증·개인화 추천·금융 데이터 수집 및 LLM 정규화 | 금융상품 391건 정규화, FSS 97건 반복 실험 |
| [룰카드](#2-룰카드--결정론적-필요경비-판정) | Backend | 규칙 엔진·영속화·API 계약·협업 및 배포 자동화 | 백엔드 단위 테스트 80개 통과 |
| [Kyverno Governance Platform](#3-kyverno-governance-platform) | Backend | 인증·RBAC와 정책 예외의 승인·적용·재시도·만료 흐름 | 77개 스위트·617개 테스트 통과 |

## AI 협업 방식

- AI가 만든 구현과 테스트도 의도를 검증하지 못하면 사용하지 않습니다. 자연어 프롬프트를 단순 문자열 포함 여부로 확인하던 테스트는 작은 문구 수정에도 깨져 삭제했습니다.
- 비결정적 에이전트 동작을 확인하려고 평가 하네스를 만들었고, 과도한 토큰 사용이 확인되자 반복 횟수를 줄이고 LLM-as-a-Judge 호출을 제거해 검증 품질과 비용을 다시 조정했습니다.
- 근거: [카카오테크캠퍼스 PR #158 — AI 활용 내역·리뷰·수정 기록](https://github.com/kakaotechcampus-4/pusan-clone/pull/158)

---

## 1. Y-FIN — 청년 맞춤 금융상품 추천

> 2026.03 ~ 진행 중 · 5인 팀(백엔드 2인) · PNU 2026 AI 해커톤 최우수상
> [서비스 백엔드](https://github.com/ApptiveDev/Fin-BE) · [금융 데이터 수집기](https://github.com/ApptiveDev/Fin-API) · [해커톤 저장소](https://github.com/PNU-2026-AI-Hackathon/pnuai-c-07-finfin2)

사용자의 소득·나이·가구·거래 조건을 바탕으로 예·적금 상품을 필터링하고 적합도와 예상 수익을 제공하는 서비스입니다. 은행상품 328건과 정부 정책상품 63건, 총 391건을 정규화했으며 서비스 API와 외부 데이터 수집기를 별도 애플리케이션으로 운영합니다.

### 인증과 사용자 상태

- Google·Kakao OAuth2 로그인과 Spring Security 설정을 구성했습니다.
- JWT Access/Refresh Token 발급과 Cookie 기반 토큰 회전을 구현했습니다.
- 동시에 들어온 Refresh 요청이 토큰 상태를 깨뜨리는 문제를 수정했습니다.
- Spring Security 계층과 애플리케이션 계층의 예외 처리 책임을 분리했습니다.
- 근거: [인증 기반 구현 PR #3](https://github.com/ApptiveDev/Fin-BE/pull/3), [토큰 동시성 수정 PR #9](https://github.com/ApptiveDev/Fin-BE/pull/9), [인가 예외 처리 PR #11](https://github.com/ApptiveDev/Fin-BE/pull/11)

### 약관과 개인화 추천

- 약관을 버전과 효력 시점을 가진 도메인으로 모델링했습니다.
- 필수 최신 약관에 동의하지 않은 사용자의 권한과 재동의 흐름을 구현했습니다.
- 사용자 응답에 따라 다음 입력 항목과 기본값이 달라지는 동적 폼을 구현했습니다.
- 상품별 여러 금리 옵션을 분리하고, 가입 가능 옵션을 기준으로 적합도·금리 순위를 계산했습니다.
- 근거: [약관 버전 관리 PR #12](https://github.com/ApptiveDev/Fin-BE/pull/12), [동적 입력 폼 PR #15](https://github.com/ApptiveDev/Fin-BE/pull/15), [상품 모델 개선 PR #22](https://github.com/ApptiveDev/Fin-BE/pull/22)

### 금융 데이터 수집

- 금융감독원과 온통청년 데이터를 Spring Batch로 수집했습니다.
- 서로 다른 원본 구조를 공통 상품·옵션 모델로 정규화했습니다.
- 수집원에서 사라진 상품을 비활성화하는 동기화 단계를 구현했습니다.
- 키워드 분류를 책임별 컴포넌트와 점수 기반 판정으로 개선했습니다.
- Testcontainers와 실제 PostgreSQL을 사용하는 통합 테스트를 구성했습니다.
- 근거: [상품 동기화 PR #1](https://github.com/ApptiveDev/Fin-API/pull/1), [분류 개선 PR #3](https://github.com/ApptiveDev/Fin-API/pull/3)

### LLM 정규화 안정화

- 비정형 우대조건을 Gemini로 구조화하는 과정에서 일부 요청이 약 60초 뒤 timeout되거나 응답 형식 때문에 파싱에 실패하는 문제를 호출별 시간·성공 여부 로그로 재현했습니다.
- 로컬 FSS 원본 97건에 출력 토큰 상한과 temperature 조합을 반복 적용했습니다. 기준 설정의 전체 실행 시간 5분 30초를 선택 설정에서 2분 52초~3분 23초로 줄였고, 6회 실행에서 기존 장시간 timeout이 다시 발생하지 않았습니다.
- 포괄 조건과 세부 조건, 구간별 차등 우대금리를 구분하도록 프롬프트를 7차례 수정한 뒤 같은 97건 회귀 데이터에서 parsing failure 0건을 확인했습니다. 이 결과는 조정에 사용한 데이터의 회귀 검증 결과이며 운영 전체 성능으로 일반화하지 않습니다.
- 근거: [LLM 정규화 안정화 PR #40](https://github.com/PNU-2026-AI-Hackathon/pnuai-c-07-finfin2/pull/40), [97건 검토 자료](https://github.com/user-attachments/files/32417255/fss_normalization_v7_review.xlsx)

---

## 2. 룰카드 — 결정론적 필요경비 판정

> 카카오테크 캠퍼스 4기 · 6인 팀 프로젝트 · 진행 중
> [팀 저장소](https://github.com/kakaotechcampus-4/ktc4-pusan-4) · [아키텍처](https://github.com/kakaotechcampus-4/ktc4-pusan-4/blob/develop/docs/architecture.md) · [프론트엔드 프로토타입](https://yuyeol3.github.io/tax-agent-prototype/)

IT 프리랜서의 카드 거래가 필요경비인지 법적 근거와 함께 설명하는 서비스입니다. AI는 규칙 후보와 근거 초안을 만들고, 실제 판정은 버전 관리되는 룰 엔진이 담당합니다.

### 규칙 엔진

- 차단형 규칙과 속성 누적형 규칙을 6개 관문으로 구분했습니다.
- 우선순위·구체성·ID에 따른 규칙 승자 결정 방식을 코드로 고정했습니다.
- 잘못된 필드 타입, 근거 없는 확정 판정, 충돌하는 속성 선언을 로딩 시점에 차단했습니다.
- 사용자 답변이 충돌하면 배치 전체가 아닌 해당 거래만 `확인 필요`로 전환했습니다.
- 근거: [룰 엔진 PR #10](https://github.com/kakaotechcampus-4/ktc4-pusan-4/pull/10), [구조·영속화 개선 PR #9](https://github.com/kakaotechcampus-4/ktc4-pusan-4/pull/9)

### 영속화와 동시성

- 룰·법령 버전을 판정 결과와 함께 저장해 결과의 재현 근거를 남겼습니다.
- PostgreSQL 잠금으로 사용자별 버전·개정 번호 발급을 직렬화했습니다.
- 판정 도중 발생한 개별 데이터 충돌이 전체 배치를 중단하지 않도록 실패 범위를 제한했습니다.
- 근거: [판정 DB·영속화 PR #11](https://github.com/kakaotechcampus-4/ktc4-pusan-4/pull/11)

### API 계약과 개발 자동화

- 문서화된 API 계약을 실행 가능한 NestJS 목 서버로 구현했습니다.
- 업로드 멱등성, 판정 실행, 되묻기, 수동 오버라이드 흐름을 테스트했습니다.
- 정적 분석·테스트 결과를 PR에 요약하는 코드 품질 봇을 추가했습니다.
- Discord PR 알림·리뷰 리마인드와 ECR·EC2 배포 흐름을 구성했습니다.
- 근거: [목 서버 PR #41](https://github.com/kakaotechcampus-4/ktc4-pusan-4/pull/41), [코드 품질 봇 PR #37](https://github.com/kakaotechcampus-4/ktc4-pusan-4/pull/37), [리뷰 알림 PR #48](https://github.com/kakaotechcampus-4/ktc4-pusan-4/pull/48), [AWS 배포 PR #58](https://github.com/kakaotechcampus-4/ktc4-pusan-4/pull/58)

### 실행 검증

- 현재 `develop` 기준 백엔드 단위 테스트 **80개 통과**를 직접 확인했습니다.
- 별도 프론트엔드 프로토타입은 테스트 6개, lint, production build가 통과합니다.
- 상호명 184만 개를 이용한 분류 실험에서 규칙+모델 조합을 평가하고, 정확도뿐 아니라 공개 데이터에 없는 카테고리와 도메인 시프트 한계를 기록했습니다.
- 근거: [상호명 분류 실험](https://github.com/yuyeol3/merchant-category-classifier), [평가 보고서](https://github.com/yuyeol3/merchant-category-classifier/blob/main/REPORT.md)

---

## 3. Kyverno Governance Platform

> 부산대학교 졸업과제 · 3인 · Backend 담당
> [저장소](https://github.com/pnucse-capstone2026/capstone-2026-team-30) · [시연 영상](https://www.youtube.com/watch?v=Gngvrf_XGBg)

![Kyverno Governance Platform 관리자 화면](https://raw.githubusercontent.com/pnucse-capstone2026/capstone-2026-team-30/main/docs/images/admin-dashboard.jpg)

Kubernetes 정책 위반을 조회하고, 한시적 예외의 요청·승인·적용·만료·감사 과정을 관리하는 플랫폼입니다.

### 인증과 접근 제어

- JWT Access/Refresh Token 인증과 세션 회전을 구현했습니다.
- 로그아웃과 토큰 갱신의 동시 요청에서도 세션 상태가 일관되도록 트랜잭션을 보강했습니다.
- `ADMIN`, `APPROVER`, `REQUESTER`, `VIEWER` RBAC와 사용자별 클러스터 접근 범위를 적용했습니다.
- 근거: [인증 구현](https://github.com/pnucse-capstone2026/capstone-2026-team-30/commit/3878df473a4936d06b0aec4c1ae7226f05e1347c), [RBAC 구현](https://github.com/pnucse-capstone2026/capstone-2026-team-30/commit/cbfe42f1e46fc222f1c2ed7e357fe879347e77dd), [세션 동시성 개선](https://github.com/pnucse-capstone2026/capstone-2026-team-30/commit/f355ff132fdfc03c49dbc9eb3764fedb3f44eda3)

### 정책 예외 상태 관리

- 정책 위반·예외 처리 API, 감사 로그, Kubernetes `PolicyException` 연동을 구현했습니다.
- 승인 결정과 원격 클러스터 적용 결과를 별도 상태로 관리했습니다.
- Kubernetes API의 일시적 실패를 지수 백오프와 조정 루프로 재시도했습니다.
- 예외 만료·취소 시 CR을 회수하고, 실제 상태가 DB와 어긋나면 다시 수렴하도록 구성했습니다.
- 근거: [예외 조정 개선](https://github.com/pnucse-capstone2026/capstone-2026-team-30/commit/95dd69ed2138529689e6dd0ac107d4adcea5582d), [동시성 개선](https://github.com/pnucse-capstone2026/capstone-2026-team-30/commit/6018d01b3da5c5d488618c29a3db21b491925c9b)

### 테스트 및 성능 검증

- 현재 `main` 기준 백엔드 **77개 테스트 스위트, 617개 테스트 통과**를 직접 확인했습니다.
- 팀 성능 평가에서 동일 요청 20건 동시 승인 시 1건만 승인되고 중복 CR은 생성되지 않았습니다.
- 원격 EKS 예외 적용은 p50 225.11ms, EKS 위반 조회는 50 VU에서 96.79 RPS·오류율 0%로 측정했습니다.
- 측정 환경과 한계: [테스트·성능 평가](https://github.com/pnucse-capstone2026/capstone-2026-team-30#45-테스트-및-성능-평가)

---

## 기술 스택

| 구분 | 경험 |
| --- | --- |
| Backend | Java, Spring Boot, Spring Security, Spring Batch, NestJS, REST API, WebSocket |
| Data | PostgreSQL, MySQL, Redis, JPA, Prisma, Flyway, 트랜잭션·잠금 |
| Test | JUnit, Jest, Supertest, Testcontainers, 계약·통합·동시성 테스트 |
| Infra | AWS EC2·ECR·RDS·EKS, Docker Compose, GitHub Actions, Kubernetes, Kyverno |
| Frontend | TypeScript, React, Next.js, Vite |
| AI / Data | 구조화 출력 검증, RAG, scikit-learn, char n-gram 분류 |

## 기타 프로젝트

| 프로젝트 | 내용 | 상태 |
| --- | --- | --- |
| [Plato Calendar](https://github.com/yuyeol3/plato-calendar3) | 부산대학교 LMS 일정을 수집·동기화하는 Chrome Extension | production build 확인 |
| [YouTube Shortener](https://github.com/yuyeol3/youtube-shortener-backend) | 시청 heatmap 기반 인기 구간 탐색·자동 스킵 웹 서비스 | Spring Boot + React 풀스택 |
| [Yacht Online](https://github.com/yuyeol3/yacht-online) | WebSocket 멀티플레이 게임, API/Game 서버 분리와 AWS 배포 | Spring Boot + Redis + AWS |
| [개발 블로그](https://yuyeol3.github.io/) | Next.js App Router와 GitHub Actions 기반 정적 블로그 | GitHub Pages 운영 |
| [Codex 개발 도구](https://github.com/yuyeol3/maintain-code-map) | 코드맵, TDD, 변경 설명 등 반복 개발 작업을 구조화한 도구 | [관련 저장소](https://github.com/yuyeol3?tab=repositories) |
