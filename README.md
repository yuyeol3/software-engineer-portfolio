<!-- markdownlint-disable MD013 -->

# 최유렬 | Software Engineer Portfolio

[![GitHub](https://img.shields.io/badge/GitHub-yuyeol3-181717?logo=github)](https://github.com/yuyeol3)
[![Java](https://img.shields.io/badge/Java-Spring%20Boot-6DB33F?logo=springboot&logoColor=white)](#기술-스택)
[![TypeScript](https://img.shields.io/badge/TypeScript-NestJS-3178C6?logo=typescript&logoColor=white)](#기술-스택)

CJ올리브네트웍스 Software Engineer 지원 포트폴리오

백엔드에서 생기는 동시성 문제와 상태 전이 오류를 재현하고 테스트로 확인해 왔습니다. Java와 Spring으로 서비스 및 데이터 파이프라인을 구현했습니다. LLM을 쓸 때는 출력 검증과 실패 처리 기준을 먼저 정합니다.

## 직무 연관 경험

| 공고의 업무 및 우대사항 | 관련 경험 |
| --- | --- |
| S/W 설계 및 개발 | 금융상품 추천 / 실시간 게임 서버 / Kubernetes 정책 관리 시스템의 데이터 모델 및 API 구현 |
| WEB 서비스 개발 및 운영 | 인증 및 인가 / 트랜잭션 / 외부 API 연동 / 실시간 상태 동기화 / 배포 |
| Java / Spring Boot | OAuth2/JWT 인증 / 개인화 검색 / Spring Batch 수집기 / WebSocket 게임 서버 |
| 클라우드 및 데이터 | EC2 배포 / RDS 연동 / PostgreSQL / Redis / 잠금과 동시성 제어 |
| LLM 및 생성형 AI 연동 | Y-FIN의 Gemini 정규화 파이프라인 및 일정관리 에이전트의 RAG와 MCP 연동 |
| AI 협업 역량 | 생성 코드의 취약한 테스트를 폐기하고 평가 하네스의 검증 범위와 토큰 비용을 조정한 기록 |

## 프로젝트 요약

| 프로젝트 | 역할 | 핵심 기여 | 대표 검증 |
| --- | --- | --- | --- |
| [Y-FIN](#1-y-fin-청년-맞춤-금융상품-추천) | Backend / Data Pipeline | 인증 / 개인화 추천 / 금융 데이터 수집 / LLM 정규화 | 금융상품 391건 정규화 및 FSS 97건 반복 실험 |
| [Yacht Online](#2-yacht-online-실시간-멀티플레이-게임) | Backend / Game Server / EC2 | API와 Game 서버 분리 / Redis 기반 서버 선택 / WebSocket 상태 동기화 | API / Game / Redis 통합 실행 후 Game 서버 2대 AWS 배포 |
| [Kyverno Governance Platform](#3-kyverno-governance-platform) | Backend | 인증과 RBAC / 정책 예외 승인 / 적용 및 재시도 / 만료 처리 | 77개 테스트 스위트에서 617개 테스트 통과 |

## AI 협업 방식

- AI가 만든 구현과 테스트도 의도를 검증하지 못하면 사용하지 않습니다. 자연어 프롬프트를 단순 문자열 포함 여부로 확인하던 테스트는 작은 문구 수정에도 깨져 삭제했습니다.
- 비결정적 에이전트 동작을 확인하려고 평가 하네스를 만들었습니다. 과도한 토큰 사용이 확인된 뒤 반복 횟수를 줄였습니다. LLM-as-a-Judge 호출도 제거해 검증 품질과 비용을 다시 조정했습니다.
- 근거: [카카오테크캠퍼스 PR #158: AI 활용 내역과 리뷰 기록](https://github.com/kakaotechcampus-4/pusan-clone/pull/158)

---

## 1. Y-FIN: 청년 맞춤 금융상품 추천

> 2026.03 ~ 진행 중 / 5인 팀(백엔드 2인) / PNU 2026 AI 해커톤 최우수상
> [서비스 백엔드](https://github.com/ApptiveDev/Fin-BE) / [금융 데이터 수집기](https://github.com/ApptiveDev/Fin-API) / [해커톤 저장소](https://github.com/PNU-2026-AI-Hackathon/pnuai-c-07-finfin2)

사용자의 소득과 나이 그리고 가구 조건을 바탕으로 예금과 적금 상품을 필터링합니다. 거래 조건에 따른 적합도와 예상 수익도 제공합니다. 은행상품 328건과 정부 정책상품 63건을 합친 391건을 정규화했습니다. 서비스 API와 외부 데이터 수집기는 별도 애플리케이션으로 운영합니다.

### 인증과 사용자 상태

- Google과 Kakao OAuth2 로그인 및 Spring Security 설정을 구성했습니다.
- JWT Access/Refresh Token 발급과 Cookie 기반 토큰 회전을 구현했습니다.
- 동시에 들어온 Refresh 요청이 토큰 상태를 깨뜨리는 문제를 수정했습니다.
- Spring Security 계층과 애플리케이션 계층의 예외 처리 책임을 분리했습니다.
- 근거: [인증 기반 구현 PR #3](https://github.com/ApptiveDev/Fin-BE/pull/3) / [토큰 동시성 수정 PR #9](https://github.com/ApptiveDev/Fin-BE/pull/9) / [인가 예외 처리 PR #11](https://github.com/ApptiveDev/Fin-BE/pull/11)

### 약관과 개인화 추천

- 약관을 버전과 효력 시점을 가진 도메인으로 모델링했습니다.
- 필수 최신 약관에 동의하지 않은 사용자의 권한과 재동의 흐름을 구현했습니다.
- 사용자 응답에 따라 다음 입력 항목과 기본값이 달라지는 동적 폼을 구현했습니다.
- 상품별 여러 금리 옵션을 분리했습니다. 가입 가능한 옵션을 기준으로 적합도와 금리 순위를 계산했습니다.
- 근거: [약관 버전 관리 PR #12](https://github.com/ApptiveDev/Fin-BE/pull/12) / [동적 입력 폼 PR #15](https://github.com/ApptiveDev/Fin-BE/pull/15) / [상품 모델 개선 PR #22](https://github.com/ApptiveDev/Fin-BE/pull/22)

### 금융 데이터 수집

- 금융감독원과 온통청년 데이터를 Spring Batch로 수집했습니다.
- 서로 다른 원본 구조를 공통 상품과 옵션 모델로 정규화했습니다.
- 수집원에서 사라진 상품을 비활성화하는 동기화 단계를 구현했습니다.
- 키워드 분류를 책임별 컴포넌트와 점수 기반 판정으로 개선했습니다.
- Testcontainers와 실제 PostgreSQL을 사용하는 통합 테스트를 구성했습니다.
- 근거: [상품 동기화 PR #1](https://github.com/ApptiveDev/Fin-API/pull/1) / [분류 개선 PR #3](https://github.com/ApptiveDev/Fin-API/pull/3)

### LLM 정규화 안정화

- 비정형 우대조건을 Gemini로 구조화하는 과정에서 일부 요청이 약 60초 뒤 timeout됐습니다. 응답 형식 때문에 파싱에 실패하는 문제도 호출별 소요 시간과 성공 여부 로그로 재현했습니다.
- 로컬 FSS 원본 97건에 출력 토큰 상한과 temperature 조합을 반복 적용했습니다. 기준 설정의 전체 실행 시간은 5분 30초였습니다. 선택한 설정에서는 2분 52초에서 3분 23초가 걸렸습니다. 6회 실행 중 기존의 장시간 timeout은 다시 발생하지 않았습니다.
- 포괄 조건과 세부 조건을 구분하고 구간별 차등 우대금리도 따로 처리하도록 프롬프트를 7차례 수정했습니다. 같은 97건 회귀 데이터에서 parsing failure 0건을 확인했습니다. 이 결과는 조정에 사용한 데이터의 회귀 검증 결과이며 운영 전체 성능을 뜻하지 않습니다.
- 근거: [LLM 정규화 안정화 PR #40](https://github.com/PNU-2026-AI-Hackathon/pnuai-c-07-finfin2/pull/40) / [97건 검토 자료](https://github.com/user-attachments/files/32417255/fss_normalization_v7_review.xlsx)

---

## 2. Yacht Online: 실시간 멀티플레이 게임

> 2026.02 ~ 2026.06 / 3인 팀 / Backend / Game Server / Redis / EC2 담당
> [통합 저장소](https://github.com/yuyeol3/yacht-online) / [백엔드 저장소](https://github.com/yuyeol3/yacht-backend/tree/feat/seperate-game-api) / [시스템 구성도](https://github.com/yuyeol3/yacht-online/blob/main/docs/images/diagram.png)

![Yacht Online 시스템 구성도](https://raw.githubusercontent.com/yuyeol3/yacht-online/main/docs/images/diagram.png)

로컬에서 동작하던 Yacht Dice 게임을 최대 4명이 참여하는 실시간 웹 서비스로 확장하고 AWS에 배포했습니다. 하나의 Spring Boot 코드베이스를 REST API 서버와 WebSocket Game 서버 역할로 나눴습니다. Game 서버는 2대로 구성했습니다.

### API와 Game 서버 분리

- 실행 설정과 조건부 컴포넌트로 하나의 코드베이스를 API와 Game 역할로 나눴습니다. 각 서버에는 필요한 기능만 활성화했습니다.
- Redis Sorted Set에 서버별 방 개수를 기록했습니다. API 서버는 방이 가장 적은 Game 서버 중 응답 가능한 서버를 선택합니다.
- Game 서버가 10초마다 heartbeat을 갱신합니다. 서버 키에는 30초 TTL을 적용해 갱신이 끊긴 서버를 선택 후보에서 제외했습니다.
- 방과 Game 서버의 연결을 Redis에 저장해 이후 요청이 같은 서버로 전달되도록 했습니다.
- 근거: [API와 Game 서버 분리 커밋](https://github.com/yuyeol3/yacht-backend/commit/eb6515c) / [GameServerRegistryService](https://github.com/yuyeol3/yacht-backend/blob/865dd4056a69ab2da8875b1a9194779c79333096/src/main/java/io/github/yuyeol3/yachtbackend/server/GameServerRegistryService.java)

### 실시간 게임 상태 동기화

- STOMP 메시징으로 방 단위 상태를 동기화했습니다. 주사위 굴리기와 고정 해제를 서버에서 검증했습니다. 점수 선택과 턴 전환도 검증한 뒤 참가자에게 브로드캐스트했습니다.
- 최대 4명 참여와 3분 턴 제한을 서버 규칙으로 두고 게임 상태를 관리했습니다.
- Redis는 서버 레지스트리와 방 배정에 사용했습니다. 게임 진행 상태는 Game 서버가 관리하도록 책임을 구분했습니다.
- 근거: [GameRoomWebSocketController](https://github.com/yuyeol3/yacht-backend/blob/865dd4056a69ab2da8875b1a9194779c79333096/src/main/java/io/github/yuyeol3/yachtbackend/gameroom/GameRoomWebSocketController.java) / [WebSocket 연결 종료 처리](https://github.com/yuyeol3/yacht-backend/blob/865dd4056a69ab2da8875b1a9194779c79333096/src/main/java/io/github/yuyeol3/yachtbackend/config/WebSocketEventListener.java)

### 클라우드 배포

- Docker Compose에서 API 서버와 Game 서버가 Redis를 통해 함께 동작하는지 확인했습니다. 이후 EC2에 배포하고 RDS와 ALB를 연동했습니다.
- 본인은 백엔드 구조 개선과 Game 서버 2대 구성 그리고 Redis와 EC2 배포를 담당했습니다. 다른 팀원은 RDS와 S3 정적 호스팅을 맡았습니다. ACM 인증서와 DNS 및 ALB 구성도 해당 팀원이 담당했습니다.
- 이 경험의 범위는 클라우드 환경 구축과 배포입니다. 장기 운영이나 모니터링 경험으로 확대해 표현하지 않습니다.
- 근거: [통합 저장소 README](https://github.com/yuyeol3/yacht-online#readme) / [백엔드 실행과 배포 구성](https://github.com/yuyeol3/yacht-backend/tree/feat/seperate-game-api)

---

## 3. Kyverno Governance Platform

> 부산대학교 졸업과제 / 3인 / Backend 담당
> [저장소](https://github.com/pnucse-capstone2026/capstone-2026-team-30) / [시연 영상](https://www.youtube.com/watch?v=Gngvrf_XGBg)

![Kyverno Governance Platform 관리자 화면](https://raw.githubusercontent.com/pnucse-capstone2026/capstone-2026-team-30/main/docs/images/admin-dashboard.jpg)

Kubernetes 정책 위반을 조회합니다. 한시적 예외가 요청부터 승인과 적용을 거쳐 만료되기까지의 과정과 감사 기록을 관리하는 플랫폼입니다.

### 인증과 접근 제어

- JWT Access/Refresh Token 인증과 세션 회전을 구현했습니다.
- 로그아웃과 토큰 갱신의 동시 요청에서도 세션 상태가 일관되도록 트랜잭션을 보강했습니다.
- `ADMIN` / `APPROVER` / `REQUESTER` / `VIEWER` RBAC와 사용자별 클러스터 접근 범위를 적용했습니다.
- 근거: [인증 구현](https://github.com/pnucse-capstone2026/capstone-2026-team-30/commit/3878df473a4936d06b0aec4c1ae7226f05e1347c) / [RBAC 구현](https://github.com/pnucse-capstone2026/capstone-2026-team-30/commit/cbfe42f1e46fc222f1c2ed7e357fe879347e77dd) / [세션 동시성 개선](https://github.com/pnucse-capstone2026/capstone-2026-team-30/commit/f355ff132fdfc03c49dbc9eb3764fedb3f44eda3)

### 정책 예외 상태 관리

- 정책 위반과 예외 처리 API를 구현했습니다. 감사 로그와 Kubernetes `PolicyException` 연동도 맡았습니다.
- 승인 결정과 원격 클러스터 적용 결과를 별도 상태로 관리했습니다.
- Kubernetes API의 일시적 실패를 지수 백오프와 조정 루프로 재시도했습니다.
- 예외가 만료되거나 취소되면 CR을 회수합니다. 실제 상태가 DB와 어긋난 경우 다시 수렴하도록 구성했습니다.
- 근거: [예외 조정 개선](https://github.com/pnucse-capstone2026/capstone-2026-team-30/commit/95dd69ed2138529689e6dd0ac107d4adcea5582d) / [동시성 개선](https://github.com/pnucse-capstone2026/capstone-2026-team-30/commit/6018d01b3da5c5d488618c29a3db21b491925c9b)

### 테스트 및 성능 검증

- 현재 `main` 기준 백엔드 **77개 테스트 스위트에서 617개 테스트 통과**를 직접 확인했습니다.
- 팀 성능 평가에서 동일 요청 20건 동시 승인 시 1건만 승인되고 중복 CR은 생성되지 않았습니다.
- 원격 EKS 예외 적용은 p50 225.11ms였습니다. EKS 위반 조회는 50 VU에서 96.79 RPS를 기록했고 오류율은 0%였습니다.
- 측정 환경과 한계: [테스트 및 성능 평가](https://github.com/pnucse-capstone2026/capstone-2026-team-30#45-테스트-및-성능-평가)

---

## 기술 경험

- 주로 사용: Java / Spring Boot / Spring Security / JPA / PostgreSQL
- 프로젝트에서 사용: Spring Batch / Redis / Docker / AWS EC2 / RDS
- 특정 프로젝트에서 사용: NestJS / Kubernetes / Kyverno / RAG / MCP

기술별 사용 범위는 각 프로젝트 설명에 따로 적었습니다.

## 기타 프로젝트

| 프로젝트 | 내용 | 상태 |
| --- | --- | --- |
| [Plato Calendar](https://github.com/yuyeol3/plato-calendar3) | 부산대학교 LMS 일정을 수집하고 동기화하는 Chrome Extension | production build 확인 |
| [YouTube Shortener](https://github.com/yuyeol3/youtube-shortener-backend) | 시청 heatmap 기반 인기 구간 탐색과 자동 스킵 웹 서비스 | Spring Boot + React 풀스택 |
| [Kanana Schedule Agent](https://github.com/kakaotechcampus-4/pusan-clone/tree/choiyuyeol/final) | 도구 선택부터 RAG와 MCP 연동 그리고 하위 에이전트 위임까지 6주간 구현 | Python + RAG + MCP |
| [개발 블로그](https://yuyeol3.github.io/) | Next.js App Router와 GitHub Actions 기반 정적 블로그 | GitHub Pages 운영 |
| [Codex 개발 도구](https://github.com/yuyeol3/maintain-code-map) | 코드맵과 TDD 그리고 변경 설명처럼 반복되는 개발 작업을 구조화한 도구 | [관련 저장소](https://github.com/yuyeol3?tab=repositories) |
