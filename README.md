<!-- markdownlint-disable MD013 -->

# 최유렬 | Software Engineer Portfolio

[![GitHub](https://img.shields.io/badge/GitHub-yuyeol3-181717?logo=github)](https://github.com/yuyeol3)
[![Java](https://img.shields.io/badge/Java-Spring%20Boot-6DB33F?logo=springboot&logoColor=white)](#기술-경험)

CJ올리브네트웍스 Software Engineer 지원 포트폴리오

- 부산대학교 정보컴퓨터공학 2027.02 졸업예정 (학점 4.36/4.5, 전공 4.42/4.5)
- zerom50@gmail.com
- 수상: PNU 2026 AI 해커톤 최우수상(2026.08), 카카오테크캠퍼스 4기 아이디어톤 우수상(2026.07), 제4회 PNU Coding Challenge 장려상(2024.11)
- 자격: 정보처리기사(2026.09), OPIc IH

백엔드에서 발생하는 동시성 문제와 상태 전이 오류를 직접 재현하고 테스트 코드로 검증해 왔습니다. Java와 Spring을 기반으로 웹 서비스 및 데이터 파이프라인을 설계하고 구현했습니다. LLM을 연동할 때는 비결정적 동작에 대비해 출력 검증과 실패 처리 기준을 먼저 정의합니다.

## 직무 연관 경험

| 공고의 업무 및 우대사항 | 관련 경험 |
| --- | --- |
| S/W 설계 및 개발 | 금융상품 추천, 실시간 게임 서버, Kubernetes 정책 관리 시스템의 데이터 모델과 API 구현 |
| WEB 서비스 개발 및 운영 | 인증과 인가, 트랜잭션, 외부 API 연동, 실시간 상태 동기화, 배포 |
| Java / Spring Boot | OAuth2/JWT 인증, 추천 로직 리팩터링, Spring Batch 수집기, WebSocket 게임 서버 |
| 클라우드 및 데이터 | EC2 배포, RDS 연동, PostgreSQL, Redis, 잠금과 동시성 제어 |
| LLM 및 생성형 AI 연동 | Y-FIN의 Gemini 정규화 파이프라인, 일정관리 에이전트의 RAG와 MCP 연동 |
| AI 협업 역량 | [일정관리 AI 에이전트](#4-일정관리-ai-에이전트-카카오테크캠퍼스-4기)의 평가 하네스 기반 AI 개발 루프, AI 생성 테스트 선별 및 폐기, 검증 비용 조정 |

## 프로젝트 요약

| 프로젝트 | 역할 | 핵심 기여 | 대표 검증 |
| --- | --- | --- | --- |
| [Y-FIN](#1-y-fin-청년-맞춤-금융상품-추천) | Backend, Data Pipeline | 인증, 금융 데이터 수집, LLM 정규화, 추천 로직 리팩터링 | 금융상품 391건 정규화 및 FSS 97건 반복 실험 |
| [Yacht Online](#2-yacht-online-실시간-멀티플레이-게임) | Backend, Game Server, EC2 | API와 Game 서버 분리, Redis 기반 서버 선택, WebSocket 상태 동기화 | API, Game, Redis 통합 로컬 검증 및 Game 서버 2대 AWS EC2 분산 배포 |
| [Kyverno Governance Platform](#3-kyverno-governance-platform) | Backend (인증, 세션, 정책 예외) | 인증과 RBAC, 세션 동시성, 예외 조정 작업의 중복 처리 방지 | 로그아웃과 동시 refresh 경합 통합 테스트 |
| [일정관리 AI 에이전트](#4-일정관리-ai-에이전트-카카오테크캠퍼스-4기) | 개인 구현 (Python) | 도구 라우팅, 듀얼 RAG, MCP 연동, 하위 에이전트 위임 | 도구 호출 trace 기반 평가 하네스와 held-out 케이스 |

---

## 1. Y-FIN: 청년 맞춤 금융상품 추천

> 2026.03 ~ 진행 중 / 5인 팀(백엔드 2인) / PNU 2026 AI 해커톤 최우수상\
> 기술: Java 21, Spring Boot 4, Spring Security, Spring Batch, JPA, PostgreSQL, Testcontainers, Gemini API\
> [서비스 백엔드](https://github.com/ApptiveDev/Fin-BE) / [금융 데이터 수집기](https://github.com/ApptiveDev/Fin-API) / [해커톤 저장소](https://github.com/PNU-2026-AI-Hackathon/pnuai-c-07-finfin2)

사용자의 소득, 연령, 가구 조건을 바탕으로 예금과 적금 상품을 맞춤 필터링하고, 거래 조건에 따른 개인별 적합도와 예상 수익을 산출해 주는 서비스입니다. 은행 상품 328건과 정부 정책 상품 63건 등 총 391건의 금융 데이터를 수집하고 정규화했습니다. 서비스 API와 외부 데이터 수집기는 독립된 애플리케이션으로 분리 운영합니다.

**담당 범위:** 인증과 인가, 금융 데이터 수집기 신규 구축, 추천 로직 리팩터링과 우대금리 매칭을 맡았습니다. 검색과 정렬의 최초 설계는 다른 백엔드 팀원이 담당했습니다.

### 인증과 사용자 상태

- Google 및 Kakao OAuth2 소셜 로그인과 Spring Security 기반 인증 체계를 구축했습니다.
- JWT Access/Refresh Token 발급 및 HttpOnly Cookie 기반의 토큰 회전(Rotation)을 구현했습니다.
- **Refresh 동시성:** 같은 Refresh Token으로 요청이 동시에 들어오면 두 요청이 모두 토큰을 유효하다고 읽고 회전을 시도해 500 오류가 발생했습니다. 행 삭제 방식을 "활성 상태인 행만 비활성화하는 조건부 UPDATE"로 바꾸고 영향 행 수를 확인해, 먼저 도달한 요청만 통과하고 나머지는 401로 처리되게 했습니다.
- 벌크 UPDATE가 영속성 컨텍스트를 비우면서 이후 사용자 조회에 영향을 주는 문제도 확인해, 사용자 조회를 갱신보다 앞에 두었습니다.
- Spring Security 필터 계층과 애플리케이션 서비스 계층의 예외 처리 책임을 명확히 분리했습니다.
- 근거: [인증 기반 구현 PR #3](https://github.com/ApptiveDev/Fin-BE/pull/3) / [토큰 동시성 수정 PR #9](https://github.com/ApptiveDev/Fin-BE/pull/9) / [인가 예외 처리 PR #11](https://github.com/ApptiveDev/Fin-BE/pull/11)

### 약관과 개인화 추천

- 약관 변경 이력을 추적할 수 있도록 버전과 효력 시점을 갖는 독립 도메인으로 모델링했습니다.
- 최신 필수 약관에 동의하지 않은 사용자의 접근 권한을 제한하고 재동의를 유도하는 흐름을 구현했습니다.
- 사용자의 응답에 따라 후속 입력 항목과 기본값이 동적으로 결정되는 폼 로직을 구현했습니다.
- 상품별로 다양한 우대금리 옵션을 분리 모델링하고, 가입 가능한 옵션 기준으로 적합도와 금리 순위를 산출했습니다.
- 나이와 비대면가입 우대금리가 사용자 조건과 관계없이 매칭되던 문제를 조건 검증 로직으로 차단했습니다.
- 중복된 상품 조건 판정, 사용자 입력 정책, 최고이율 판정을 공통 정책 클래스로 분리하고, 목록과 상세 응답이 같은 정책을 쓰는지 통합 테스트로 검증했습니다.
- 근거: [약관 버전 관리 PR #12](https://github.com/ApptiveDev/Fin-BE/pull/12) / [동적 입력 폼 PR #15](https://github.com/ApptiveDev/Fin-BE/pull/15) / [상품 모델 개선 PR #22](https://github.com/ApptiveDev/Fin-BE/pull/22) / [우대금리 매칭 수정](https://github.com/PNU-2026-AI-Hackathon/pnuai-c-07-finfin2/commit/2aa792c946) / [추천 정책 분리](https://github.com/PNU-2026-AI-Hackathon/pnuai-c-07-finfin2/commit/7cfa89ba86)

### 금융 데이터 수집

- 금융감독원 오픈API와 온통청년 정책 데이터를 Spring Batch로 수집했습니다.
- 출처마다 상이한 원본 데이터 구조를 공통 상품 및 옵션 표준 스키마로 정규화했습니다.
- 수집 대상에서 제외되거나 마감된 상품을 감지해 안전하게 비활성화하는 동기화 배치를 구현했습니다.
- 단순 텍스트 매칭 방식의 키워드 분류를 단일 책임 컴포넌트 분리와 점수(Score) 기반 판정 로직으로 리팩토링했습니다.
- Testcontainers 기반의 실제 PostgreSQL 환경에서 배치 잡과 데이터 정합성을 검증하는 통합 테스트를 구축했습니다.
- 근거: [상품 동기화 PR #1](https://github.com/ApptiveDev/Fin-API/pull/1) / [분류 개선 PR #3](https://github.com/ApptiveDev/Fin-API/pull/3)

### LLM 정규화 안정화

![Y-FIN LLM 보강 정규화 흐름](https://raw.githubusercontent.com/PNU-2026-AI-Hackathon/pnuai-c-07-finfin2/main/docs/img/llm-normalization.png)

- **실패 격리:** 규칙으로 파싱되지 않는 약관과 우대조건만 Gemini로 보강합니다. 호출이나 검증에 실패하면 규칙 기반 정규화 결과를 그대로 사용하고, 실패 횟수와 시각을 기록해 6시간 동안 같은 항목을 다시 호출하지 않도록 했습니다.
- 비정형 텍스트로 된 우대조건을 Gemini로 구조화할 때, 일부 요청이 약 60초간 지연된 후 timeout되거나 응답 형식 이상으로 파싱에 실패하는 문제를 호출별 소요 시간과 성공 여부 로깅을 통해 재현했습니다.
- 로컬에 구축한 금융감독원(FSS) 원본 97건 데이터셋을 대상으로 출력 토큰 상한(Max Tokens)과 Temperature 조합을 반복 실험했습니다. 그 결과, 기준 설정 기준 5분 30초였던 전체 실행 시간을 2분 52초~3분 23초로 약 38~48% 단축했으며, 6회 반복 실행 동안 장시간 timeout이 단 1건도 재발하지 않았습니다.
- 포괄 조건과 세부 조건을 명확히 구분하고 구간별 차등 우대금리도 정확히 처리할 수 있도록 프롬프트를 7차례 점진적으로 고도화했습니다. 조정에 사용한 동일한 97건 회귀 데이터셋에서 파싱 실패(Parsing Failure) 0건을 확인했습니다. 새 데이터에서의 일반화는 아직 별도로 확인하지 않았습니다.
- 근거: [실패 격리 구현](https://github.com/PNU-2026-AI-Hackathon/pnuai-c-07-finfin2/blob/63cc1965a7e22b5864298bd69cc70df9f6f5e008/api/src/main/java/apptive/fin/apicollector/normalize/enrich/FssLlmProductDraftEnricher.java) / [LLM 정규화 안정화 PR #40](https://github.com/PNU-2026-AI-Hackathon/pnuai-c-07-finfin2/pull/40) / [97건 검토 자료](https://github.com/user-attachments/files/32417255/fss_normalization_v7_review.xlsx)

---

## 2. Yacht Online: 실시간 멀티플레이 게임

> 2026.02 ~ 2026.06 / 3인 팀 / Backend, Game Server, Redis, EC2 담당\
> 기술: Java, Spring Boot, WebSocket(STOMP), Redis, Docker Compose, AWS EC2, RDS, ALB\
> [통합 저장소](https://github.com/yuyeol3/yacht-online) / [백엔드 저장소](https://github.com/yuyeol3/yacht-backend) / [시스템 구성도](https://github.com/yuyeol3/yacht-online/blob/main/docs/images/diagram.png)

![Yacht Online 시스템 구성도](https://raw.githubusercontent.com/yuyeol3/yacht-online/main/docs/images/diagram.png)

로컬 환경에서 동작하던 Yacht Dice 게임을 최대 4인 참여가 가능한 실시간 웹 서비스로 확장하고 AWS 환경에 배포했습니다. 단일 Spring Boot 코드베이스를 유지하면서 런타임 설정에 따라 REST API 서버와 WebSocket Game 서버로 역할을 분리하고, Game 서버를 2대로 수평 확장할 수 있도록 설계했습니다.

### API와 Game 서버 분리

- 실행 프로파일과 조건부 컴포넌트(`@ConditionalOnProperty`)를 활용해 단일 코드베이스에서 API와 Game 서버 역할을 분리 구동하고, 인스턴스별로 필요한 컴포넌트만 활성화했습니다.
- Redis Sorted Set에 서버별 방 개수를 기록하고, API 서버가 방 생성 시 heartbeat 키가 남아 있는 Game 서버 중 방이 가장 적은 서버를 선택해 배정하도록 했습니다.
- 각 Game 서버가 10초마다 heartbeat를 갱신하고 Redis 키에 30초 TTL을 부여하여, 갱신이 중단된 비정상 서버를 라우팅 후보에서 자동으로 제외했습니다.
- 생성된 방과 담당 Game 서버의 매핑 정보를 Redis에 저장하여, 이후의 클라이언트 요청이 일관되게 동일한 Game 서버로 라우팅되도록 보장했습니다.
- 근거: [API와 Game 서버 분리 커밋](https://github.com/yuyeol3/yacht-backend/commit/eb6515c) / [GameServerRegistryService](https://github.com/yuyeol3/yacht-backend/blob/865dd4056a69ab2da8875b1a9194779c79333096/src/main/java/io/github/yuyeol3/yachtbackend/server/GameServerRegistryService.java)

### 실시간 게임 상태 동기화

- Spring WebSocket과 STOMP 프로토콜을 사용해 방 단위의 실시간 게임 상태를 동기화했습니다.
- 클라이언트 조작에 의한 변조를 방지하기 위해 주사위 굴리기, 주사위 고정/해제(Keep), 점수 등록, 턴 전환 등 모든 게임 로직을 서버에서 검증한 후 참가자들에게 브로드캐스트했습니다.
- 방당 최대 4인 참여 제한과 턴당 3분 제한 시간을 서버 규칙으로 관리했습니다.
- Redis는 서버 레지스트리와 방 배정 메타데이터 관리에 사용하고, 게임 진행 상태는 Game 서버 메모리에서 직접 관리하도록 역할을 분리했습니다.
- **한계:** 진행 상태가 Game 서버 메모리에 있어서, 서버가 내려가면 그 서버에서 진행 중이던 게임은 복구되지 않습니다. 새로 만드는 방은 남아 있는 서버로 배정됩니다.
- 근거: [GameRoomWebSocketController](https://github.com/yuyeol3/yacht-backend/blob/865dd4056a69ab2da8875b1a9194779c79333096/src/main/java/io/github/yuyeol3/yachtbackend/gameroom/GameRoomWebSocketController.java) / [WebSocket 연결 종료 처리](https://github.com/yuyeol3/yacht-backend/blob/865dd4056a69ab2da8875b1a9194779c79333096/src/main/java/io/github/yuyeol3/yachtbackend/config/WebSocketEventListener.java)

### 클라우드 배포

- Docker Compose를 활용해 API 서버, 멀티 Game 서버, Redis 간의 연동을 로컬에서 통합 검증한 후 AWS EC2 환경에 배포하고 RDS 및 ALB를 연동했습니다.
- 역할 분담: 본인은 백엔드 아키텍처 개선, Game 서버 2대 구성, Redis 연동 및 EC2 배포를 주도했습니다. 다른 팀원들은 RDS 구성, S3 프론트엔드 정적 호스팅, ACM SSL 인증서, DNS와 ALB 설정을 담당했습니다.
- 근거: [통합 저장소 README](https://github.com/yuyeol3/yacht-online#readme) / [백엔드 저장소](https://github.com/yuyeol3/yacht-backend)

---

## 3. Kyverno Governance Platform

> 부산대학교 졸업과제 / 3인 / Backend 담당 (인증과 인가, 세션, 정책 예외 API와 상태 전이, Kyverno 어댑터)\
> 기술: TypeScript, NestJS 11, Prisma 6, PostgreSQL, Kubernetes Client, Kyverno\
> [저장소](https://github.com/pnucse-capstone2026/capstone-2026-team-30) / [시연 영상](https://www.youtube.com/watch?v=Gngvrf_XGBg)

![Kyverno Governance Platform 관리자 화면](https://raw.githubusercontent.com/pnucse-capstone2026/capstone-2026-team-30/main/docs/images/admin-dashboard.jpg)

Kubernetes 클러스터의 정책 위반 사항을 모니터링하고, 한시적 예외(PolicyException)의 신청부터 승인, 배포 적용, 만료 및 회수에 이르는 전 과정과 감사 기록을 체계적으로 관리하는 거버넌스 플랫폼입니다.

### 인증과 접근 제어

- JWT Access/Refresh Token 인증 체계와 보안 강화를 위한 세션 회전(Rotation)을 구현했습니다.
- **세션 동시성:** 로그아웃과 토큰 갱신이 동시에 들어오면 로그아웃 뒤에도 활성 Refresh Token이 남을 수 있었습니다. 사용자별 토큰 폐기를 Serializable 트랜잭션으로 묶고, 이 경합 시나리오를 통합 테스트로 고정했습니다.
- 역할 기반 접근 제어(`ADMIN`, `APPROVER`, `REQUESTER`, `VIEWER` RBAC)를 설계하고 사용자 권한에 따른 클러스터 접근 범위를 제한했습니다.
- 근거: [인증 구현](https://github.com/pnucse-capstone2026/capstone-2026-team-30/commit/3878df473a4936d06b0aec4c1ae7226f05e1347c) / [RBAC 구현](https://github.com/pnucse-capstone2026/capstone-2026-team-30/commit/cbfe42f1e46fc222f1c2ed7e357fe879347e77dd) / [세션 동시성 개선](https://github.com/pnucse-capstone2026/capstone-2026-team-30/commit/f355ff132fdfc03c49dbc9eb3764fedb3f44eda3)

### 정책 예외 상태 관리

- 정책 위반 및 예외 처리 비즈니스 API를 구현하고, 감사 로그 기록과 Kubernetes `PolicyException` CRD 연동을 구현했습니다.
- 플랫폼의 승인 결정과 원격 클러스터의 실제 적용 결과를 독립된 상태로 분리하여 관리했습니다.
- **조정 작업 중복 방지:** 예외를 클러스터에 반영하는 조정(Reconciliation) 작업이 같은 예외를 동시에 처리하지 않도록 claim(lease)으로 대상을 선점하게 했습니다. 취소와 적용이 겹치는 경우는 Serializable 트랜잭션과 현재 상태를 조건으로 한 갱신으로 처리했습니다.
- 만료된 예외를 먼저 처리하도록 조정 순서를 바꿨습니다. 예외 기간 만료 또는 취소 시 원격 CR을 회수하며, 실제 클러스터 상태가 DB와 불일치할 경우 정상 상태로 자동 수렴하도록 구성했습니다.
- 근거: [예외 조정 개선](https://github.com/pnucse-capstone2026/capstone-2026-team-30/commit/95dd69ed2138529689e6dd0ac107d4adcea5582d) / [동시성 개선](https://github.com/pnucse-capstone2026/capstone-2026-team-30/commit/6018d01b3da5c5d488618c29a3db21b491925c9b)

### 테스트 및 성능 검증

- 현재 `main` 기준 팀 백엔드 전체 **77개 테스트 스위트, 617개 테스트 통과**를 직접 확인했습니다.
- 팀 성능 평가에서 동일 예외 건에 대해 20건의 동시 승인 요청을 보냈을 때 단 1건만 정상 승인되고 중복 CR 생성이 차단됨을 확인했습니다.
- 원격 EKS 클러스터 대상 예외 적용 지연 시간은 p50 225.11ms를 기록했으며, EKS 위반 내역 조회는 50 VU 부하에서 오류율 0%, 96.79 RPS를 기록했습니다.
- 측정 환경과 한계: [테스트 및 성능 평가](https://github.com/pnucse-capstone2026/capstone-2026-team-30#45-테스트-및-성능-평가)

---

## 4. 일정관리 AI 에이전트: 카카오테크캠퍼스 4기

> 2026.06 ~ 2026.08 / 에이전틱 AI 과정 / 개인 구현 / 6주 과제 PR 전부 병합\
> 기술: Python 3.11, LangChain, OpenAI API, ChromaDB, SQLite, MCP, pytest\
> [저장소(choiyuyeol/final)](https://github.com/kakaotechcampus-4/pusan-clone/tree/choiyuyeol/final) / [4주차 PR #125](https://github.com/kakaotechcampus-4/pusan-clone/pull/125) / [5주차 PR #158](https://github.com/kakaotechcampus-4/pusan-clone/pull/158) / [6주차 PR #195](https://github.com/kakaotechcampus-4/pusan-clone/pull/195)

자연어 대화로 일정을 관리하는 에이전트를 6주에 걸쳐 단계적으로 구현했습니다. 매주 요구사항이 늘어나, LLM이 도구를 고르는 구조에서 시작해 여러 저장소 라우팅, MCP 연동, 하위 에이전트 위임까지 확장했습니다.

### 에이전트 구조

- 질문 유형에 따라 에이전트가 ChromaDB(참고 자료 검색)와 SQLite(구조화된 일정 조회) 도구를 스스로 선택하도록 라우팅 규칙을 구성했습니다.
- MCP 서버를 연동해 멤버와 질의 기반으로 과거 대화를 검색하는 도구를 추가했습니다.
- supervisor가 하위 에이전트에게 작업을 위임하고, 약속 시간과 결정 이유를 함께 설명하도록 구성했습니다.
- 주차가 늘며 프롬프트 지시가 서로 충돌하자, 예시를 더 넣는 대신 도구에 종속된 지시는 도구 설명으로 옮기고 나머지는 필요한 것만 포함되도록 분리했습니다.

### 평가 하네스

- 에이전트 실행 trace에서 도구 호출 여부, 순서, 인자를 케이스 기대값과 대조해 채점했습니다. 앞 도구의 결과가 다음 도구의 인자로 제대로 전달됐는지도 검사했습니다.
- 같은 케이스를 여러 번 실행해 통과율로 판정했습니다. 규칙마다 프롬프트 예시에 넣지 않은 held-out 케이스를 두어, 프롬프트가 평가 케이스에만 맞춰지지 않도록 했습니다.
- 최종 답변은 검토 기준 문서에 따라 PASS, FAIL, REVIEW로 판정했습니다. 근거 없는 사실을 지어내거나 빈 검색 결과를 기록 부재로 과장하면 FAIL입니다.
- 실행 기록에서 조회 결과를 다음 도구로 넘기지 않는 문제, 빈 결과를 "가능한 시간 없음"으로 잘못 해석하는 문제를 찾아 도구 설명에 데이터 전달 규칙을 명시했습니다.
- 운영 매니저에게서 "테스트 하네스를 구현한 수강생은 유일하다"는 평가를 받았습니다.
- 근거: [trace 채점 함수](https://github.com/kakaotechcampus-4/pusan-clone/blob/2d265874262dae9f3b6e2bd68ed398c83b3420fe/tests/evals/predicates.py) / [답변 검토 기준](https://github.com/kakaotechcampus-4/pusan-clone/blob/2d265874262dae9f3b6e2bd68ed398c83b3420fe/tests/evals/ANSWER_REVIEW.md) / [운영 매니저 코멘트](https://github.com/kakaotechcampus-4/pusan-clone/pull/158#issuecomment-5161731817)

### AI와 협업한 방식

- Codex와 Claude가 평가 하네스를 실행하며 프롬프트를 반복 개선하도록 개발 루프를 구성했습니다. 사람이 결과를 눈으로 확인하는 대신 하네스 통과 여부가 개선의 기준이 됐습니다.
- 이 루프에서 input token이 과도하게 쓰인다는 지적을 받았습니다. 모든 케이스를 5회씩 돌려 80% 이상으로 판정하던 방식을 대표 케이스만 3회 중 2회 통과로 판정하고 나머지는 1회만 돌리도록 바꿨습니다. GPT-4.1 mini API로 호출하던 LLM-as-a-Judge도 폐기하고, 저장된 실행 결과를 Codex와 Claude CLI가 같은 검토 기준으로 판정하게 했습니다.
- AI가 만든 결과물도 의도를 검증하지 못하면 채택하지 않았습니다. AI가 작성한 테스트 중 자연어 프롬프트를 단순 문자열 포함 여부로 확인하던 테스트는 사소한 문구 변경에도 깨져 삭제했습니다.
- Codex가 만든 변환 함수에 입력 검증 책임까지 몰려 있는 것을 발견하고, 검증은 호출부가 맡도록 다시 지시해 책임을 분리했습니다.
- 근거: [3주차 PR #92](https://github.com/kakaotechcampus-4/pusan-clone/pull/92) / [5주차 PR #158: AI 활용 내역과 리뷰 기록](https://github.com/kakaotechcampus-4/pusan-clone/pull/158) / [LLM Judge 하네스](https://github.com/kakaotechcampus-4/pusan-clone/blob/2d265874262dae9f3b6e2bd68ed398c83b3420fe/tests/evals/llm_judge.py)

---

## 기술 경험

- 주로 사용: Java, Spring Boot, Spring Security, JPA, PostgreSQL
- 프로젝트에서 사용: Spring Batch, Redis, Docker, AWS EC2, RDS
- 특정 프로젝트에서 사용: NestJS, Kubernetes, Kyverno, RAG, MCP

기술별 사용 범위는 각 프로젝트 설명에 상세히 기술했습니다.

## 기타 프로젝트

| 프로젝트 | 내용 | 비고 |
| --- | --- | --- |
| [Plato Calendar](https://github.com/yuyeol3/plato-calendar3) | 부산대학교 LMS 일정을 자동으로 수집하고 동기화하는 Chrome Extension | production build 확인 |
| [YouTube Shortener](https://github.com/yuyeol3/youtube-shortener-backend) | 시청 Heatmap 기반 인기 구간 탐색 및 자동 스킵 웹 서비스 | Spring Boot + React 풀스택 |
| [개발 블로그](https://yuyeol3.github.io/) | Next.js App Router와 GitHub Actions 기반 정적 기술 블로그 | GitHub Pages 운영 |
| [Codex 개발 도구](https://github.com/yuyeol3/maintain-code-map) | 코드맵, TDD, 변경 설명 등 반복되는 개발 생산성 작업을 구조화한 도구 | [관련 저장소](https://github.com/yuyeol3?tab=repositories) |
