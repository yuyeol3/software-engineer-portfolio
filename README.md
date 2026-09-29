<!-- markdownlint-disable MD013 -->

# 최유렬 | Software Engineer Portfolio

[![GitHub](https://img.shields.io/badge/GitHub-yuyeol3-181717?logo=github)](https://github.com/yuyeol3)
[![Java](https://img.shields.io/badge/Java-Spring%20Boot-6DB33F?logo=springboot&logoColor=white)](#기술-경험)

CJ올리브네트웍스 Software Engineer 지원 포트폴리오

- 부산대학교 정보컴퓨터공학 2027.02 졸업예정 (학점 4.36/4.5, 전공 4.42/4.5)
- zerom50@gmail.com
- 수상: PNU 2026 AI 해커톤 최우수상(2026.08), 카카오테크캠퍼스 4기 아이디어톤 우수상(2026.07), 제4회 PNU Coding Challenge 장려상(2024.11)
- 자격: 정보처리기사(2026.09), OPIc IH

불편한 것을 코드로 빠르게 해결하는 재미로 코딩을 시작했습니다. 지금은 전공에서 학습한 데이터베이스, 운영체제 개념 등을 많이 활용해볼 수 있는 Java와 Spring 백엔드를 중심으로 개발하고 있습니다. 최근에는 AI로 개발하며 코드 검증과 이해에 관심이 있습니다.

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
| [Y-FIN](#1-y-fin-청년-맞춤-금융상품-추천) | Backend, Data Pipeline | 인증, 금융 데이터 수집, LLM 정규화, 추천 로직 리팩터링 | FSS 97건 반복 실험으로 실행 시간 38\~48% 단축 |
| [Yacht Online](#2-yacht-online-실시간-멀티플레이-게임) | Backend, Game Server, EC2 | API와 Game 서버 분리, Redis 기반 서버 선택, WebSocket 상태 동기화 | API, Game, Redis 통합 로컬 검증 및 Game 서버 2대 AWS EC2 분산 배포 |
| [Kyverno Governance Platform](#3-kyverno-governance-platform) | Backend (인증, 세션, 정책 예외) | 인증과 RBAC, 정책 예외 상태 전이와 조정 루프, 동시성 제어 | 로그아웃과 동시 refresh 경합 통합 테스트 |
| [일정관리 AI 에이전트](#4-일정관리-ai-에이전트-카카오테크캠퍼스-4기) | 개인 구현 (Python) | 도구 라우팅, 듀얼 RAG, MCP 연동, 하위 에이전트 위임 | 도구 호출 trace 기반 평가 하네스와 held-out 케이스 |

---

## 1. Y-FIN: 청년 맞춤 금융상품 추천

> 2026.03 \~ 진행 중 / 5인 팀(백엔드 2인) / PNU 2026 AI 해커톤 최우수상\
> 담당: 인증과 인가, 금융 데이터 수집기 신규 구축, 추천 로직 리팩터링\
> 기술: Java 21, Spring Boot 4, Spring Security, Spring Batch, JPA, PostgreSQL, Testcontainers, Gemini API\
> [서비스 백엔드](https://github.com/ApptiveDev/Fin-BE) / [금융 데이터 수집기](https://github.com/ApptiveDev/Fin-API) / [해커톤 저장소](https://github.com/PNU-2026-AI-Hackathon/pnuai-c-07-finfin2)

금융감독원과 온통청년의 서로 다른 상품 데이터를 하나의 스키마로 정규화하고, 소득, 연령, 가구 조건으로 가입 가능한 상품을 걸러 예상 수익을 계산하는 서비스입니다.

**1. 동시 Refresh 요청의 500 오류**\
같은 Refresh Token으로 요청이 동시에 들어오면 두 요청이 모두 토큰을 유효하다고 읽고 회전을 시도해 500 오류가 났습니다. 행 삭제를 "활성 상태인 행만 비활성화하는 조건부 UPDATE"로 바꾸고 영향 행 수를 확인해, 먼저 온 요청만 통과하고 나머지는 401로 처리되게 했습니다. [PR #9](https://github.com/ApptiveDev/Fin-BE/pull/9)

**2. LLM 호출 실패로 수집이 중단되고 재호출이 반복되던 문제**\
규칙으로 파싱되지 않는 약관과 우대조건만 Gemini로 보강하도록 했습니다. 호출이나 검증에 실패하면 규칙 기반 결과를 그대로 쓰고, 실패한 항목은 6시간 동안 다시 호출하지 않습니다. [구현 코드](https://github.com/PNU-2026-AI-Hackathon/pnuai-c-07-finfin2/blob/63cc1965a7e22b5864298bd69cc70df9f6f5e008/api/src/main/java/apptive/fin/apicollector/normalize/enrich/FssLlmProductDraftEnricher.java)

![Y-FIN LLM 보강 정규화 흐름](https://raw.githubusercontent.com/PNU-2026-AI-Hackathon/pnuai-c-07-finfin2/main/docs/img/llm-normalization.png)

**3. LLM 장시간 timeout과 파싱 실패**\
일부 요청이 약 60초 뒤 timeout되고, 응답 형식 때문에 파싱이 실패했습니다. 호출별 소요 시간 로그로 재현한 뒤 금융감독원(FSS) 원본 97건으로 출력 토큰 상한과 temperature 조합을 실험해, 전체 실행 시간을 5분 30초에서 2분 52초\~3분 23초로 줄였습니다(약 38\~48%). 6회 실행 중 장시간 timeout은 재발하지 않았습니다. 프롬프트를 7차례 고친 뒤 같은 97건에서 파싱 실패 0건을 확인했고, 조정에 쓴 데이터라 새 데이터에서의 일반화는 아직 확인하지 않았습니다. [PR #40](https://github.com/PNU-2026-AI-Hackathon/pnuai-c-07-finfin2/pull/40) / [97건 검토 자료](https://github.com/user-attachments/files/32417255/fss_normalization_v7_review.xlsx)

<details>
<summary>그 외 구현</summary>

- Google, Kakao OAuth2 로그인과 JWT Access/Refresh Token, HttpOnly Cookie 기반 토큰 회전을 구현했습니다.
- 조건부 UPDATE 도입 후 벌크 UPDATE가 영속성 컨텍스트를 비워 사용자 조회에 영향을 주는 문제를 확인해, 조회를 갱신보다 앞에 두었습니다.
- Spring Security 필터 계층과 애플리케이션 서비스 계층의 예외 처리 책임을 분리했습니다.
- 약관을 버전과 효력 시점을 갖는 도메인으로 모델링하고, 최신 필수 약관 미동의 사용자의 권한 제한과 재동의 흐름을 구현했습니다.
- 사용자 응답에 따라 후속 입력 항목과 기본값이 바뀌는 동적 폼을 구현했습니다.
- 나이와 비대면가입 우대금리가 사용자 조건과 관계없이 매칭되던 문제를 수정했습니다. 상품 조건 판정과 최고이율 판정을 공통 정책 클래스로 분리하고, 목록과 상세 응답의 일관성을 통합 테스트로 확인했습니다.
- 금융감독원과 온통청년 데이터를 Spring Batch로 수집해 공통 상품 스키마로 정규화하고, 수집원에서 사라진 상품을 비활성화하는 동기화 단계를 구현했습니다.
- 키워드 분류를 책임별 컴포넌트와 점수 기반 판정으로 개선하고, Testcontainers와 실제 PostgreSQL로 통합 테스트를 구성했습니다.
- 근거: [인증 PR #3](https://github.com/ApptiveDev/Fin-BE/pull/3) / [인가 예외 처리 PR #11](https://github.com/ApptiveDev/Fin-BE/pull/11) / [약관 버전 관리 PR #12](https://github.com/ApptiveDev/Fin-BE/pull/12) / [동적 입력 폼 PR #15](https://github.com/ApptiveDev/Fin-BE/pull/15) / [상품 모델 개선 PR #22](https://github.com/ApptiveDev/Fin-BE/pull/22) / [우대금리 매칭 수정](https://github.com/PNU-2026-AI-Hackathon/pnuai-c-07-finfin2/commit/2aa792c946) / [추천 정책 분리](https://github.com/PNU-2026-AI-Hackathon/pnuai-c-07-finfin2/commit/7cfa89ba86) / [상품 동기화 PR #1](https://github.com/ApptiveDev/Fin-API/pull/1) / [분류 개선 PR #3](https://github.com/ApptiveDev/Fin-API/pull/3)

</details>

---

## 2. Yacht Online: 실시간 멀티플레이 게임

> 2026.02 \~ 2026.06 / 3인 팀\
> 담당: Backend, Game Server, Redis, EC2 배포\
> 기술: Java, Spring Boot, WebSocket(STOMP), Redis, Docker Compose, AWS EC2, RDS, ALB\
> [통합 저장소](https://github.com/yuyeol3/yacht-online) / [백엔드 저장소](https://github.com/yuyeol3/yacht-backend)

![Yacht Online 시스템 구성도](https://raw.githubusercontent.com/yuyeol3/yacht-online/main/docs/images/diagram.png)

로컬에서만 동작하던 Yacht Dice 게임을 최대 4명이 함께 하는 실시간 웹 서비스로 확장하고, Game 서버 2대로 AWS에 배포했습니다.

**1. 여러 Game 서버 중 살아 있는 서버로 방 배정**\
Game 서버를 2대로 늘리자 어느 서버로 방을 보낼지, 죽은 서버를 어떻게 뺄지를 정해야 했습니다. Game 서버가 10초마다 Redis에 heartbeat를 갱신하고 키에 30초 TTL을 두어, API 서버는 키가 남아 있는 서버 중 방이 가장 적은 서버(Sorted Set)를 고릅니다. 방과 서버의 매핑도 Redis에 저장해 이후 요청이 같은 서버로 갑니다. [GameServerRegistryService](https://github.com/yuyeol3/yacht-backend/blob/865dd4056a69ab2da8875b1a9194779c79333096/src/main/java/io/github/yuyeol3/yachtbackend/server/GameServerRegistryService.java)

**2. 하나의 코드베이스로 API 서버와 Game 서버 분리**\
실행 프로파일과 `@ConditionalOnProperty`로 역할마다 필요한 컴포넌트만 켜서, 코드베이스를 나누지 않고 REST API 서버와 WebSocket Game 서버로 구동했습니다. [분리 커밋](https://github.com/yuyeol3/yacht-backend/commit/eb6515c)

**3. 서버가 판정하는 실시간 동기화와 그 한계**\
STOMP로 방 단위 상태를 동기화하고, 주사위 굴리기, 고정과 해제, 점수 선택, 턴 전환을 모두 서버에서 검증한 뒤 브로드캐스트했습니다. 진행 상태는 Game 서버 메모리에 두었기 때문에, 서버가 내려가면 그 서버에서 진행 중이던 게임은 복구되지 않습니다. 새로 만드는 방은 남아 있는 서버로 배정됩니다. [GameRoomWebSocketController](https://github.com/yuyeol3/yacht-backend/blob/865dd4056a69ab2da8875b1a9194779c79333096/src/main/java/io/github/yuyeol3/yachtbackend/gameroom/GameRoomWebSocketController.java)

<details>
<summary>그 외 구현</summary>

- 방당 최대 4명 참여와 턴당 3분 제한을 서버 규칙으로 관리했습니다.
- Docker Compose로 API 서버, Game 서버 2대, Redis의 연동을 로컬에서 검증한 뒤 EC2에 배포하고 RDS, ALB와 연동했습니다.
- 근거: [WebSocket 연결 종료 처리](https://github.com/yuyeol3/yacht-backend/blob/865dd4056a69ab2da8875b1a9194779c79333096/src/main/java/io/github/yuyeol3/yachtbackend/config/WebSocketEventListener.java) / [통합 저장소 README](https://github.com/yuyeol3/yacht-online#readme) / [시스템 구성도](https://github.com/yuyeol3/yacht-online/blob/main/docs/images/diagram.png)

</details>

---

## 3. Kyverno Governance Platform

> 부산대학교 졸업과제 / 3인\
> 담당: Backend (인증과 인가, 세션, 정책 예외 API와 상태 전이, reconciler, Kyverno 어댑터)\
> 기술: TypeScript, NestJS 11, Prisma 6, PostgreSQL, Kubernetes Client, Kyverno\
> [저장소](https://github.com/pnucse-capstone2026/capstone-2026-team-30) / [시연 영상](https://www.youtube.com/watch?v=Gngvrf_XGBg)

![Kyverno Governance Platform 관리자 화면](https://raw.githubusercontent.com/pnucse-capstone2026/capstone-2026-team-30/main/docs/images/admin-dashboard.jpg)

Kubernetes 정책 위반을 조회하고, 한시적 정책 예외의 신청, 승인, 클러스터 적용, 만료와 회수까지의 과정과 감사 기록을 관리하는 플랫폼입니다.

**1. 승인 결정과 클러스터 반영이 어긋나는 문제: 중간 상태를 둔 상태 전이**\
승인은 DB에서 끝나도 클러스터 반영은 실패할 수 있어서, 반영 중인 중간 상태(`APPLYING`, `CANCELLING`, `EXPIRING`)를 따로 두었습니다. 승인되면 `APPLYING`이 되고 CR 적용이 확인되어야 `APPROVED`가 됩니다. 모든 전이는 Serializable 트랜잭션 안에서 "현재 상태가 기대한 값일 때만" 갱신하고 전후 상태를 감사 로그에 남깁니다. 팀 성능 평가에서 같은 예외에 20건을 동시에 승인 요청했을 때 1건만 승인되고 중복 CR이 생기지 않았습니다. [상태 전이 서비스](https://github.com/pnucse-capstone2026/capstone-2026-team-30/blob/b2aec85fc78d4a9a8d51ceb1632b66c02761456b/apps/backend/src/exception-lifecycle/exception-lifecycle.service.ts)

**2. 적용 실패와 DB 불일치를 다시 맞추는 조정 루프**\
주기적으로 도는 reconciler가 중간 상태에 머문 예외를 다시 적용하거나 회수하고, 승인된 예외의 CR이 남아 있는지 확인합니다. 이후 여러 조정 작업이 같은 예외를 동시에 처리하지 않도록 claim(lease)으로 대상을 선점하게 했습니다. [reconciler](https://github.com/pnucse-capstone2026/capstone-2026-team-30/blob/b2aec85fc78d4a9a8d51ceb1632b66c02761456b/apps/backend/src/exception-lifecycle/exception-reconciler.service.ts) / [중복 처리 방지 커밋](https://github.com/pnucse-capstone2026/capstone-2026-team-30/commit/6018d01b3da5c5d488618c29a3db21b491925c9b)

**3. 로그아웃과 토큰 갱신의 경합**\
로그아웃과 토큰 갱신이 동시에 들어오면 로그아웃 뒤에도 활성 Refresh Token이 남을 수 있었습니다. 사용자별 토큰 폐기를 Serializable 트랜잭션으로 묶고, 이 경합 시나리오를 통합 테스트로 고정했습니다. [세션 동시성 수정](https://github.com/pnucse-capstone2026/capstone-2026-team-30/commit/f355ff132fdfc03c49dbc9eb3764fedb3f44eda3)

<details>
<summary>그 외 구현과 검증</summary>

- JWT Access/Refresh Token 인증과 세션 회전, `ADMIN`, `APPROVER`, `REQUESTER`, `VIEWER` RBAC와 사용자별 클러스터 접근 범위를 구현했습니다.
- 정책 예외 요청, 승인, 반려 API와 예외를 `PolicyException` manifest로 변환해 적용하는 Kyverno 어댑터를 만들었습니다.
- 만료된 예외를 먼저 처리하도록 조정 순서를 바꿨습니다.
- 팀 성능 평가에서 원격 EKS 예외 적용은 p50 225.11ms, 위반 조회는 50 VU에서 96.79 RPS, 오류율 0%였습니다.
- 근거: [인증 구현](https://github.com/pnucse-capstone2026/capstone-2026-team-30/commit/3878df473a4936d06b0aec4c1ae7226f05e1347c) / [RBAC 구현](https://github.com/pnucse-capstone2026/capstone-2026-team-30/commit/cbfe42f1e46fc222f1c2ed7e357fe879347e77dd) / [Kyverno 어댑터](https://github.com/pnucse-capstone2026/capstone-2026-team-30/blob/b2aec85fc78d4a9a8d51ceb1632b66c02761456b/apps/backend/src/kubernetes/kyverno.adapter.ts) / [예외 조정 개선](https://github.com/pnucse-capstone2026/capstone-2026-team-30/commit/95dd69ed2138529689e6dd0ac107d4adcea5582d) / [테스트 및 성능 평가](https://github.com/pnucse-capstone2026/capstone-2026-team-30#45-테스트-및-성능-평가)

</details>

---

## 4. 일정관리 AI 에이전트: 카카오테크캠퍼스 4기

> 2026.06 \~ 2026.08 / 에이전틱 AI 과정 / 개인 구현 / 6주 과제 PR 전부 병합\
> 기술: Python 3.11, LangChain, OpenAI API, ChromaDB, SQLite, MCP, pytest\
> [저장소(choiyuyeol/final)](https://github.com/kakaotechcampus-4/pusan-clone/tree/choiyuyeol/final) / [4주차 PR #125](https://github.com/kakaotechcampus-4/pusan-clone/pull/125) / [5주차 PR #158](https://github.com/kakaotechcampus-4/pusan-clone/pull/158) / [6주차 PR #195](https://github.com/kakaotechcampus-4/pusan-clone/pull/195)

자연어 대화로 일정을 관리하는 에이전트를 6주에 걸쳐 구현했습니다. 도구 선택에서 시작해 ChromaDB와 SQLite 라우팅, MCP 연동, 하위 에이전트 위임까지 확장했습니다.

**1. 비결정적인 에이전트 동작을 자동으로 판정하는 평가 하네스**\
에이전트 실행 trace에서 도구 호출 여부, 순서, 인자와 도구 사이의 데이터 전달을 케이스 기대값과 대조해 채점했습니다. 같은 케이스를 여러 번 돌려 통과율로 판정하고, 규칙마다 프롬프트 예시에 넣지 않은 held-out 케이스를 두었습니다. 최종 답변은 근거 없는 사실을 지어내면 FAIL이 되는 기준 문서로 판정했습니다. 운영 매니저에게서 "테스트 하네스를 구현한 수강생은 유일하다"는 평가를 받았습니다. [trace 채점 함수](https://github.com/kakaotechcampus-4/pusan-clone/blob/2d265874262dae9f3b6e2bd68ed398c83b3420fe/tests/evals/predicates.py) / [답변 검토 기준](https://github.com/kakaotechcampus-4/pusan-clone/blob/2d265874262dae9f3b6e2bd68ed398c83b3420fe/tests/evals/ANSWER_REVIEW.md) / [운영 매니저 코멘트](https://github.com/kakaotechcampus-4/pusan-clone/pull/158#issuecomment-5161731817)

**2. 주차가 늘며 충돌하던 프롬프트 지시**\
하네스 실행 기록에서 조회 결과를 다음 도구로 넘기지 않거나 빈 결과를 "가능한 시간 없음"으로 잘못 해석하는 실패를 찾았습니다. 예시를 더 넣는 대신 도구에 종속된 지시와 데이터 전달 규칙은 도구 설명으로 옮기고, 나머지 지시는 필요한 것만 포함되도록 분리했습니다. [5주차 PR #158](https://github.com/kakaotechcampus-4/pusan-clone/pull/158)

**3. AI 개발 루프와 검증 비용 조정**\
Codex와 Claude가 평가 하네스를 실행하며 프롬프트를 반복 개선하도록 개발 루프를 구성했습니다. 이 루프로 input token이 과도하게 쓰인다는 지적을 받아, 모든 케이스를 5회씩 돌리던 방식을 대표 케이스만 3회 중 2회 통과로 판정하고 나머지는 1회만 돌리도록 바꿨습니다. GPT-4.1 mini API로 호출하던 LLM-as-a-Judge도 저장된 실행 결과를 Codex와 Claude CLI가 같은 기준으로 판정하는 방식으로 바꿨습니다. [LLM Judge 하네스](https://github.com/kakaotechcampus-4/pusan-clone/blob/2d265874262dae9f3b6e2bd68ed398c83b3420fe/tests/evals/llm_judge.py)

<details>
<summary>그 외 구현과 AI 결과물 검토</summary>

- 질문 유형에 따라 에이전트가 ChromaDB(참고 자료)와 SQLite(구조화된 일정) 도구를 선택하도록 라우팅 규칙을 구성했습니다.
- MCP 서버를 연동해 멤버와 질의 기반으로 과거 대화를 검색하는 도구를 추가했습니다.
- supervisor가 하위 에이전트에게 작업을 위임하고, 약속 시간과 결정 이유를 함께 설명하도록 구성했습니다.
- AI가 작성한 테스트 중 자연어 프롬프트를 문자열 포함 여부로 확인하던 테스트는 문구 변경에도 깨져 삭제했습니다.
- Codex가 만든 변환 함수에 입력 검증 책임까지 몰려 있는 것을 발견하고, 검증은 호출부가 맡도록 다시 지시했습니다.
- 근거: [3주차 PR #92](https://github.com/kakaotechcampus-4/pusan-clone/pull/92) / [5주차 PR #158: AI 활용 내역과 리뷰 기록](https://github.com/kakaotechcampus-4/pusan-clone/pull/158)

</details>

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
