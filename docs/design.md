# Vibe Coding Workflow Design

> AI Full-stack Engineer를 위한 커스텀 바이브코딩 워크플로우 설계 문서
> 기존 superpowers 스킬을 벤치마킹하고, 커뮤니티 도구를 참고하여 커스터마이징한 end-to-end 파이프라인

---

## 전체 워크플로우 개요

### 커맨드 체계

```
/vibe:feature "결제 시스템 추가"     → Feature 모드 (전체 파이프라인)
/vibe:hotfix "로그인 500 에러"       → Hotfix 모드 (디버깅 직행)
/vibe:refactor "auth 모듈 분리"     → Refactor 모드 (설계 → 리팩터링)
```

스킬 파일 구조:

```
.claude/skills/
  vibe/
    feature/SKILL.md      → /vibe:feature
    hotfix/SKILL.md       → /vibe:hotfix
    refactor/SKILL.md     → /vibe:refactor
```

### 모드별 Phase 경로

```
Feature:  1 → 2 → 3 → (4) → 5 → 6 → 7  (전체 파이프라인)
Hotfix:   4 → 3(수정) → 6 → 7             (최단 경로)
Refactor: 1 → 3 → 5 → 6 → 7              (계획 생략)
```

### Feature 모드 Phase 구성

```
Phase 1 — 브레인스토밍 (아이디어 → 요구사항 구체화)
Phase 2 — 계획 수립 (구현 계획 → 유저 확인 → 확정)
Phase 3 — TDD (테스트 주도 개발)
Phase 4 — 디버깅 (에러 대응, 필요 시에만)
Phase 5 — 코드 리뷰 (내부 + Codex)
Phase 6 — 검증 (최종 검증)
Phase 7 — 마무리 (커밋 & PR)
```

### Hotfix 모드

프로덕션 장애 등 긴급 수정이 필요할 때 사용한다.

```
① Phase 4 (디버깅) 직행
   → 심각도 판단 없이 Standard 프로세스로 즉시 진입
   → 원인 파악 → 유저 보고 → 방향 결정

② Phase 3 (수정)
   → 버그 재현 테스트 작성 (필수)
   → 수정 → 테스트 통과 확인

③ Phase 6 (QUICK 검증)
   → 빌드 + 관련 테스트 통과 확인

④ Phase 7 (마무리)
   → 커밋 확인 → 즉시 머지 또는 PR
```

### Refactor 모드

기능 변경 없이 코드 구조를 개선할 때 사용한다.

```
① Phase 1 (설계)
   → 리팩터링 범위와 목표 정의
   → 기능 변경이 아닌 구조 변경임을 확인

② Phase 3 (리팩터링)
   → 기존 테스트 전부 통과 확인 (먼저)
   → 리팩터링 진행
   → 기존 테스트 여전히 통과 확인 (필수)
   → 필요 시 테스트 추가

③ Phase 5-7 (리뷰/검증/마무리)
   → Feature 모드와 동일
```

> Refactor 모드에서는 Iron Law("테스트 없는 코드 삭제")가 적용되지 않는다. 이미 테스트가 있는 기존 코드를 다루기 때문이다. 대신 "기존 테스트가 깨지지 않는다"가 핵심 원칙이다.

### 워크플로우 원칙

- **주요 결정은 유저가 한다** — Claude는 분석/제안, 유저가 선택
- **단계 전환 시 /clear** — 세션을 짧게 유지
- **커밋은 항상 유저 확인** — 자동 커밋 금지 (engagement_level: high/balanced 시)
- **Agent teams 적극 활용** — 병렬 가능한 작업은 병렬로
- **스킬 적극 활용** — 모든 Phase에서 작업 수행 전, 관련 스킬이 있는지 확인하고 있으면 사용한다
- **Status 파일로 상태 관리** — 진행 상황을 `.claude/workflows/`에 기록하고, /clear 후에도 이어서 진행 가능하게 한다
- **회고** — 워크플로우 완료 후 프로세스 피드백을 기록하고 config에 반영한다

---

## Phase 1: 브레인스토밍

> 기반: `superpowers:brainstorming` (v5.0.7)
> 참고: everything-claude-code (instincts 패턴)

### 목적

아이디어를 협업적 대화를 통해 검증된 설계로 구체화한다.

### Hard Gate

> 설계가 승인되기 전에는 어떤 구현 행위(코드 작성, 스캐폴딩, 파일 생성)도 금지한다.
> 이 규칙은 프로젝트 규모와 관계없이 적용된다.

### 규모 판단 (기존 대비 추가)

프로젝트 컨텍스트 탐색 후, 변경 규모를 먼저 판단한다.

| 규모 | 기준 | 프로세스 |
|------|------|---------|
| **Small** | 단일 파일, ~50줄 이하 변경 | 축약 프로세스 (질문 1-2개 → 설계 요약 → 승인 → TDD/구현) |
| **Standard** | 여러 파일, 일반 기능 | 전체 프로세스 |
| **Large** | 멀티 서브시스템, 독립 컴포넌트 다수 | 분해 → 서브프로젝트별 Standard 사이클 |

### 체크리스트

#### Small 프로세스

1. 프로젝트 컨텍스트 탐색
2. 핵심 질문 1-2개
3. 설계 요약 (수 문장) + 유저 승인
4. TDD 또는 구현 스킬로 전환

#### Standard 프로세스

1. **프로젝트 컨텍스트 탐색** — 파일, 문서, 최근 커밋 확인
2. **비주얼 컴패니언 제안** (시각적 질문이 예상될 때만, 별도 메시지)
3. **질문 단계** — 한 번에 하나씩, 객관식 선호
4. **접근법 제안** — 근거 기반 서치 후 제시
5. **MVP 범위 정의** — 핵심 기능 vs 후속 이터레이션 분리
6. **설계 제시** — 섹션별 복잡도에 맞게, 섹션마다 유저 승인
7. **스펙 문서 작성** — 프로젝트 설정 경로 또는 `docs/specs/`에 저장, 커밋
8. **스펙 셀프리뷰** — placeholder scan, 일관성, 모호성, 스코프 검사
9. **유저 리뷰** — 유저 승인 후 다음 Phase로 전환
10. **writing-plans 스킬로 전환**

#### Large 프로세스

1. 프로젝트 컨텍스트 탐색
2. 독립 서브시스템 식별 및 분해
3. 서브시스템 간 관계와 빌드 순서 정의
4. 첫 번째 서브프로젝트부터 Standard 프로세스 적용
5. 각 서브프로젝트는 개별 spec → plan → implementation 사이클

### 질문 단계 상세

#### 기존 유지

- 목적 (purpose)
- 제약 조건 (constraints)
- 성공 기준 (success criteria)

#### 추가 영역

| 영역 | 질문 예시 |
|------|----------|
| **비기능 요구사항** | 예상 동시 사용자 수, 응답 시간 목표, 가용성 요건 |
| **보안 요건** | 인증/인가 방식, 데이터 보호 수준, 민감 정보 처리 |
| **기술 스택 검증** | 기존 코드베이스와의 정합성, 새 라이브러리 도입 여부 |
| **배포/인프라 제약** | 배포 환경, CI/CD 파이프라인, 리소스 제한 |

> 모든 질문이 매번 필요한 것은 아님. 프로젝트 성격에 맞는 질문만 선별한다.

### 접근법 제안 — 근거 기반 서치

#### 원칙

```
재료는 검증된 것만, 요리(조합/판단)는 적극적으로
```

- **근거 없는 추측 금지** — "~일 것 같다", "아마 ~" 같은 막연한 제안 금지
- **조합과 판단은 적극적** — 수집한 근거를 바탕으로 프로젝트에 맞는 최적 방안 도출 가능. 여러 패턴을 조합해서 새로운 접근법 제시 가능
- **판단의 근거를 반드시 함께 설명**

#### 서치 소스 (병렬 실행)

| 소스 | 도구 | 용도 |
|------|------|------|
| 공식 문서 | context7 | 라이브러리/프레임워크 API, 설정, 마이그레이션 가이드 |
| 실제 사례 | WebSearch | 베스트 프랙티스, 아키텍처 패턴, 비교 분석 |
| 유사 구현체 | GitHub (WebSearch/WebFetch) | 스켈레톤 프로젝트, 참고 코드, 검증된 패턴 |

#### 출처 표기

```
각 접근법에 근거 출처를 명시한다.

예시:
  접근법 A: Redis Pub/Sub 기반 실시간 메시징
  - 근거: Redis 공식 문서 Pub/Sub 섹션, GitHub xxx/realtime-chat 참고

  접근법 B: A의 이벤트 소싱 패턴 + B의 CQRS 구조 조합
  - 근거: A 프로젝트의 이벤트 소싱 패턴 + B 프로젝트의 CQRS 구조 기반 조합
```

#### 제안 수 유연화

- 명확한 최선이 있으면 → **1개 + 이유**
- 트레이드오프가 있으면 → **2-3개 + 추천**
- 항상 2-3개를 강제하지 않음

### MVP/Phase 구분 (기존 대비 추가)

접근법 선택 후, 설계 제시 전에 MVP 범위를 정의한다.

```
MVP (핵심 기능)
  → 이번에 반드시 구현할 것

Phase 2+ (후속 이터레이션)
  → 이후에 추가할 것, 현재는 인터페이스만 고려
```

이 구분이 설계와 구현 계획 모두에 반영된다.

### 후속 스킬 전환 (기존 대비 확장)

| 규모 | 전환 대상 |
|------|----------|
| Small | TDD 스킬 또는 구현 스킬로 직접 전환 |
| Standard | writing-plans 스킬 |
| Large | writing-plans 스킬 (서브프로젝트 단위) |
| 프로토타입/PoC | fast-track 경로 — 간소화된 구현 |

### 스펙 저장 경로

```
우선순위:
  1. 프로젝트 .claude/config에 지정된 경로
  2. docs/specs/YYYY-MM-DD-<topic>-design.md (기본값)
```

> 기존 `docs/superpowers/specs/` 경로는 사용하지 않음

### 기존 대비 변경 요약

| 항목 | 기존 (superpowers) | 커스텀 |
|------|-------------------|--------|
| 규모 분기 | 없음 (모든 프로젝트 동일 프로세스) | Small/Standard/Large 3단계 |
| 질문 영역 | purpose, constraints, success criteria | + 비기능, 보안, 기술 스택, 배포/인프라 |
| 접근법 제안 | 항상 2-3개 | 유연하게 1-3개, 근거 기반 서치 필수 |
| 접근법 근거 | 없음 (AI 판단에 의존) | context7 + WebSearch + GitHub 서치 |
| MVP 구분 | 없음 | MVP vs 후속 이터레이션 명시 |
| 후속 스킬 | writing-plans만 허용 | 규모에 따라 TDD/구현/writing-plans 분기 |
| 스펙 경로 | `docs/superpowers/specs/` 고정 | 프로젝트 설정 우선, `docs/specs/` 기본 |
| 접근법 제안 수 | 2-3개 고정 | 상황에 따라 1-3개 유연 |

---

## Phase 2: 계획 수립 (Writing Plans)

> 기반: `superpowers:writing-plans` (v5.0.7)
> 참고: vibe-kanban (칸반 태스크 관리)

### 목적

Phase 1에서 확정된 설계를 **"정확히 뭘, 어떤 순서로, 어떻게 만들지"** 실행 가능한 계획으로 변환한다. 제로 컨텍스트 엔지니어(또는 서브에이전트)도 따라할 수 있는 수준으로 구체적이어야 한다.

### 전체 흐름

```
① 스코프 체크
   → 멀티 서브시스템이면 계획 분리 제안

② 파일 구조 설계
   → 신규/수정 파일 목록, 각 파일 책임 정의

③ 태스크 분해
   → 2-5분 단위 바이트사이즈 스텝
   → 각 태스크에 TDD 테스트 설계 포함 (코드가 아닌 명세)
   → 태스크 간 의존성 그래프 + 병렬 가능 그룹 표시

④ 셀프리뷰
   → 스펙 커버리지, placeholder scan, 타입 일관성

⑤ 유저 확인 게이트
   → 파일 구조 요약
   → 태스크 수 & 예상 커밋 수
   → 스펙 대비 커버리지 매핑
   → 의존성 그래프
   → 체크포인트 간격 선택
   → 선택지: 승인 / 수정 요청 / 재작성

⑥ 계획 확정 & 저장
   → docs/plans/YYYY-MM-DD-<feature>.md

⑦ 실행 핸드오프
   → 서브에이전트 or 인라인 실행 선택
   → Phase 3 (TDD 실행)으로 전환
```

### 체크리스트

1. **스코프 체크** — 멀티 서브시스템이면 계획 분리, 각 계획은 독립적으로 동작하는 소프트웨어를 생산
2. **파일 구조 설계** — 태스크 분해 전에 파일 구조 먼저 확정. 명확한 책임 경계, 하나의 파일 = 하나의 책임
3. **태스크 분해** — 2-5분 단위, TDD 테스트 명세 포함, 의존성 그래프 포함
4. **셀프리뷰** — 스펙 커버리지, placeholder scan, 타입 일관성 검사
5. **유저 확인 게이트** — 계획 요약 제시, 유저 승인 필수
6. **계획 저장** — `docs/plans/YYYY-MM-DD-<feature>.md`에 저장, 커밋
7. **실행 핸드오프** — 실행 방식 선택 후 Phase 3으로 전환

### 태스크 구조

각 태스크는 **테스트 명세 + 구현 힌트**로 구성한다. 실제 코드는 Phase 3(TDD 실행)에서 작성한다.

```markdown
### Task N: [컴포넌트명]

**Files:**
- Create: `src/auth/login.ts`
- Create: `src/auth/login.test.ts`
- Modify: `src/routes/index.ts`

**테스트 설계:**
- 검증 대상: 이메일/비밀번호로 로그인 시 JWT 반환
- 입력: { email: "user@test.com", password: "valid" }
- 기대 출력: { token: string, expiresIn: number }
- 엣지 케이스: 잘못된 비밀번호, 미존재 유저, 빈 입력
- 테스트 유형: 단위 + 통합

**구현 힌트:**
- 인터페이스: `loginUser(credentials: LoginDto): Promise<AuthResponse>`
- 의존성: bcrypt, jsonwebtoken
- 에러 처리: UnauthorizedError, ValidationError

**의존성:** ← Task 2 (User 모델 완료 후)

**커밋 시 유저 확인:** 필수
```

> 기존 superpowers는 계획에 실제 코드까지 작성했으나, 커스텀에서는 명세만 작성한다.
> 이유: 계획 단계에서 코드를 확정하면 실행 시 컨텍스트 변화에 유연하게 대응하기 어렵다.

### 태스크 의존성 그래프 (기존 대비 추가)

태스크 간 의존 관계를 명시하여 병렬 실행과 크리티컬 패스를 식별한다.

```
Task 1 (DB 스키마)     ─┐
Task 2 (User 모델)     ─┤→ 병렬 가능
                        │
Task 3 (Auth 서비스)    ← Task 1, 2 완료 후
Task 4 (Auth API)      ← Task 3 완료 후
Task 5 (미들웨어)       ← Task 4 완료 후
```

병렬 가능한 태스크 그룹을 명시하여, 서브에이전트 실행 시 동시 디스패치를 최적화한다.

### 유저 확인 게이트 (기존 대비 추가)

셀프리뷰 완료 후, 실행 핸드오프 전에 반드시 유저 확인을 받는다.

**유저에게 보여줄 내용:**

```
📋 계획 요약
─────────────
파일 구조: 신규 5개, 수정 2개
태스크 수: 8개 (예상 커밋 8회)
의존성: Task 1,2 병렬 → Task 3 → Task 4,5 병렬 → Task 6-8 순차

스펙 커버리지:
  ✅ 사용자 인증 — Task 1-4
  ✅ 권한 관리 — Task 5-6
  ✅ 에러 처리 — Task 7
  ✅ 통합 테스트 — Task 8

체크포인트 간격:
  (A) 매 태스크마다
  (B) Phase 경계마다 ← 기본값
  (C) 전체 완료 후 한 번만

진행할까요? [승인 / 수정 요청 / 재작성]
```

### 단계적 확정 — 대규모 계획 (기존 대비 추가)

전체 태스크가 많을 경우 (10개 이상), 한 번에 확정하지 않고 Phase별로 나누어 확정한다.

```
Phase A (핵심 기능, Task 1-8)
  → 유저 확인 → 승인 → Phase 3에서 구현
  → 결과 보고

Phase B (확장 기능, Task 9-15)
  → Phase A 결과 반영하여 계획 조정 가능
  → 유저 확인 → 승인 → 구현

Phase C (마무리, Task 16-20)
  → 같은 흐름
```

이를 통해 초기 구현 결과를 보고 후속 계획을 조정할 수 있다.

### 플레이스홀더 금지 (기존 유지)

다음은 계획 실패이며 절대 작성하지 않는다:

- "TBD", "TODO", "implement later", "fill in details"
- "Add appropriate error handling" / "add validation"
- "Write tests for the above" (구체적 테스트 명세 없이)
- "Similar to Task N" (태스크 간 참조 금지, 반복 작성)
- 코드 변경을 설명만 하고 구체적 내용이 없는 스텝
- 정의되지 않은 타입, 함수, 메서드 참조

### 계획 저장 경로

```
우선순위:
  1. 프로젝트 .claude/config에 지정된 경로
  2. docs/plans/YYYY-MM-DD-<feature>.md (기본값)
```

> 기존 `docs/superpowers/plans/` 경로는 사용하지 않음

### 실행 핸드오프

계획 확정 후 실행 방식을 선택한다:

```
"계획이 확정되어 docs/plans/<파일명>.md에 저장되었습니다. 실행 방식을 선택해주세요:"

  (1) 서브에이전트 기반 (추천)
      — 태스크당 fresh 서브에이전트 디스패치
      — 병렬 가능 태스크는 동시 실행
      — 태스크 간 리뷰

  (2) 인라인 실행
      — 현재 세션에서 순차 실행
      — 체크포인트에서 리뷰
```

### 기존 대비 변경 요약

| 항목 | 기존 (superpowers) | 커스텀 |
|------|-------------------|--------|
| 유저 확인 | 없음 (바로 실행 선택) | 셀프리뷰 후 유저 확인 게이트 필수 |
| 태스크 내용 | 실제 코드 포함 | 테스트 명세 + 구현 힌트 (코드는 Phase 3) |
| 의존성 | Task 번호만 | 의존성 그래프 + 병렬 그룹 표시 |
| 대규모 계획 | 한 번에 확정 | Phase별 단계적 확정 |
| 체크포인트 | 고정 | 유저가 간격 선택 (매 태스크 / Phase / 완료 후) |
| 커밋 | 계획대로 자동 | 매 커밋 유저 확인 |
| 계획 저장 | `docs/superpowers/plans/` | `docs/plans/` |

---

## Phase 3: TDD (테스트 주도 개발)

> 기반: `superpowers:test-driven-development` (v5.0.7)
> 참고: tdd-guard (훅 기반 TDD 강제)

### 목적

Phase 2에서 작성한 테스트 명세를 실제 코드로 변환하며, RED→GREEN→REFACTOR 사이클로 구현한다.

### Iron Law

```
테스트 없이 작성된 프로덕션 코드는 삭제한다.
참고용으로 보지 않고, 적응시키지 않고, 삭제한다.
```

### 전체 흐름

```
① 프레임워크 자동 감지
   → 프로젝트 설정 파일로 테스트/커버리지 커맨드 결정

② Phase 2 테스트 명세 로드
   → 현재 태스크의 테스트 설계 확인

③ TDD 사이클 (태스크당 반복)
   → RED → GREEN → REFACTOR → COVERAGE

④ 체크포인트
   → Phase 2에서 선택한 간격에 따라 유저에게 진행 보고

⑤ 커밋
   → 유저 확인 후 커밋
```

### 프레임워크 자동 감지 (기존 대비 추가)

Phase 3 진입 시 프로젝트 설정 파일을 읽어서 테스트 환경을 자동 설정한다.

```
감지 소스 → 프레임워크 → 테스트 커맨드 → 커버리지 커맨드

package.json (jest)     → Jest     → npx jest          → npx jest --coverage
package.json (vitest)   → Vitest   → npx vitest run    → npx vitest run --coverage
pyproject.toml          → pytest   → pytest             → pytest --cov
Cargo.toml              → cargo    → cargo test         → cargo tarpaulin
go.mod                  → go test  → go test ./...      → go test -cover ./...
```

감지 실패 시 유저에게 직접 확인한다.

### TDD 사이클 상세

Phase 2에서 넘어온 **테스트 명세**를 기반으로 실행한다.

```
Phase 2 명세 예시:
  검증 대상: 로그인 시 JWT 반환
  입력: { email: "user@test.com", password: "valid" }
  기대 출력: { token: string, expiresIn: number }
  엣지 케이스: 잘못된 비밀번호, 미존재 유저
```

#### RED — 실패하는 테스트 작성

- 명세를 바탕으로 **하나의 행위**를 검증하는 테스트 작성
- 테스트명은 행위를 명확히 설명
- 실제 코드 사용 (mock은 불가피할 때만)

```
작성 후 반드시 실행하여 실패 확인 (MANDATORY)
  → 실패해야 함 (에러가 아닌 실패)
  → 실패 메시지가 기대한 이유인지 확인
  → 통과하면 기존 동작을 테스트하고 있으므로 테스트 수정
```

#### GREEN — 최소 구현

- 테스트를 통과시키는 **가장 단순한 코드** 작성
- 테스트가 요구하지 않는 기능 추가 금지
- 과잉 설계 금지 (YAGNI)

```
작성 후 반드시 실행하여 통과 확인 (MANDATORY)
  → 현재 테스트 + 기존 테스트 모두 통과
  → 실패하면 코드 수정 (테스트 수정 아님)
```

#### REFACTOR — 정리 (기존 대비 강화)

GREEN 이후에만 리팩터링한다.

```
체크 항목:
  - 긴 함수 (>50줄) → 분리 검토
  - 깊은 중첩 (>4레벨) → 평탄화
  - 뮤테이션 사용 → 이뮤터블 패턴 전환
  - 매직 넘버/하드코딩 → 상수 추출
  - 같은 패턴 3회 이상 → 추출 검토
  - 1-2회 중복은 허용 (과잉 추상화 방지)

리팩터링 후 테스트 재실행 필수 — 여전히 GREEN인지 확인
```

#### COVERAGE — 커버리지 게이트 (기존 대비 추가)

REFACTOR 완료 후 커버리지를 측정한다.

```
기준: 80% 이상

80% 이상 → 다음 태스크로 진행
80% 미만 → 부족한 부분에 대해 추가 TDD 사이클
           (엣지 케이스, 에러 경로 등)
```

### Spike — 탐색적 코딩 허용 경로 (기존 대비 추가)

불확실한 기술이나 접근법을 검증해야 할 때, TDD를 일시 해제하고 탐색할 수 있다.

```
① Spike 선언 — "탐색적 코딩을 시작합니다" (유저에게 알림)
② 자유롭게 구현 — TDD 규칙 일시 해제
③ 결과 정리 — 뭘 배웠는지, 어떤 접근이 유효한지 기록
④ Spike 코드 삭제 — Iron Law 적용
⑤ TDD로 재구현 — 배운 것을 바탕으로 정상 사이클 진행
```

> Spike는 예외가 아닌 **정의된 경로**다. "탐색이 필요하니 TDD를 건너뛰겠다"가 아니라, "Spike 경로를 밟겠다"고 선언하는 것.

### tdd-guard 훅 연동 (기존 대비 추가)

스킬 가이드에 더해 훅으로 물리적으로 TDD를 강제한다.

```
PreToolUse 훅:
  구현 파일 수정 시도 감지
    → 해당 파일에 대한 실패하는 테스트가 존재하는지 확인
    → 없으면 수정 차단 + "먼저 테스트를 작성하세요" 안내
```

스킬이 1차 가이드, 훅이 2차 안전망 역할을 한다.

### 합리화 방어 (기존 유지)

다음 변명은 모두 "코드 삭제, TDD로 재시작"을 의미한다:

| 변명 | 현실 |
|------|------|
| "너무 단순해서 테스트 불필요" | 단순한 코드도 깨진다. 테스트 30초면 된다. |
| "나중에 테스트 쓸게" | 나중에 쓴 테스트는 즉시 통과한다. 즉시 통과는 아무것도 증명 못한다. |
| "이미 수동 테스트 했다" | 수동은 임시적. 기록 없고 재실행 불가. |
| "삭제하면 X시간 낭비" | 매몰 비용 오류. 신뢰할 수 없는 코드를 유지하는 게 진짜 낭비. |
| "참고용으로만 보겠다" | 보면 적응시킨다. 삭제는 삭제다. |
| "먼저 탐색이 필요하다" | 좋다. Spike 경로를 선언하고 밟아라. |
| "TDD가 느리다" | 디버깅이 더 느리다. |

### 중간 체크포인트

Phase 2에서 유저가 선택한 간격에 따라 진행 보고한다.

```
보고 내용:
  - 완료된 태스크 / 전체 태스크
  - 현재 커버리지
  - 발견된 이슈 (있으면)
  - 다음 태스크 미리보기

유저 선택지:
  - 계속 진행
  - 방향 조정
  - 일시 중지
```

### 디버깅 연계

TDD 사이클 중 버그를 발견하면:

```
① 버그를 재현하는 실패하는 테스트 작성
② 테스트 실패 확인
③ 2회 이상 수정 실패 시 → Phase 4 (디버깅) 진입
④ 수정 완료 후 TDD 사이클로 복귀
```

### 기존 대비 변경 요약

| 항목 | 기존 (superpowers) | 커스텀 |
|------|-------------------|--------|
| 테스트 입력 | 스킬 내에서 자체 판단 | Phase 2 테스트 명세 기반 |
| 커버리지 | 체크 없음 | 80% 게이트 필수 |
| 프레임워크 | Jest/TS 편향 | 프로젝트 자동 감지 |
| Refactor | "중복 제거, 이름 개선" | 구조화된 체크 항목 |
| Spike | 없음 (삭제만) | 정의된 Spike 경로 |
| 강제력 | 스킬 가이드 수준 | tdd-guard 훅 이중 안전망 |
| 체크포인트 | 없음 | 유저 선택 간격으로 보고 |
| 커밋 | 계획대로 자동 | 매 커밋 유저 확인 |

---

## Phase 4: 디버깅

> 기반: `superpowers:systematic-debugging` (v5.0.7)
> 참고: Continuous-Claude-v3 (5층 코드 분석)

### 목적

버그 발생 시 추측이 아닌 증거 기반으로 근본 원인을 파악하고, **유저와 함께 해결 방향을 결정**한다.

### 철칙

```
증거 없이 수정 금지.
"아마 이것 때문일 것이다" → 금지.
"일단 고쳐보자" → 금지.
```

### 진입 조건

Phase 4는 독립적으로 진입하거나, 다른 Phase에서 자동 전환된다.

```
- Phase 3 (TDD) 중 테스트 2회 이상 수정 실패 시 → 자동 진입
- 유저가 직접 버그 리포트 시 → 직접 진입
- 빌드/배포 실패 시 → 직접 진입
```

### 심각도 기반 분기 (기존 대비 추가)

버그를 처음 접했을 때 심각도를 판단하고, 그에 맞는 프로세스를 밟는다.

#### Trivial — 명백한 원인

오타, import 누락, 세미콜론 등 원인이 즉시 보이는 경우.

```
① 원인 확인
② 수정 제안 → 유저 승인 → 적용
③ 테스트 통과 확인
```

#### Standard — 일반 버그

원인이 바로 보이지 않는 일반적인 버그.

```
① 근본 원인 조사 (자동)
   → 에러 메시지, 스택 트레이스, 관련 코드 분석
② 패턴 분석 (자동)
   → 유사 패턴, 최근 변경사항, 영향 범위 파악
③ 유저에게 보고 ← (기존 대비 앞당김)
   → 표준 보고 템플릿으로 결과 제시
   → 유저와 함께 해결 방향 결정
④ 수정 구현
   → 합의된 방향으로 수정
⑤ 검증
   → 테스트 통과 확인
```

#### Complex — 복잡한 버그

멀티 컴포넌트, 재현 불가, 간헐적 발생 등.

```
① 근본 원인 조사 시작 (자동)
② 즉시 유저에게 보고 ← (조사 초기에 유저 합류)
③ 이후 전 과정 유저와 협업
   → 가설 수립도 함께
   → 수정 방향도 함께
④ 수정 및 검증
```

### 표준 보고 템플릿 (기존 대비 추가)

유저에게 디버깅 결과를 보고할 때 다음 형식을 사용한다.

```markdown
## 디버깅 보고

**증상:** [관찰된 현상 — 에러 메시지, 예상과 다른 동작 등]

**조사 범위:** [확인한 파일, 로그, 컴포넌트 목록]

**근본 원인 후보:**
1. [후보 A] — 확신도: 높음 — 증거: [구체적 증거]
2. [후보 B] — 확신도: 중간 — 증거: [구체적 증거]

**추천 해결 방향:** [A안 설명]

**대안:** [B안 설명 (있으면)]

**유저 결정 필요:** [어떤 방향으로 갈지 / 추가 조사가 필요한지]
```

### 에스컬레이션

수정 시도가 3회 실패하면 단순 버그가 아닌 **설계 문제**를 의심한다.

```
3회 실패 시:
  → 즉시 유저에게 보고
  → "이 버그는 코드 수정이 아닌 설계 변경이 필요할 수 있습니다"
  → 선택지 제시:
    (A) 설계 변경 검토 (Phase 1로 부분 회귀)
    (B) 우회 방안 탐색
    (C) 현재 접근 계속 시도
```

### TDD 연계

디버깅은 항상 테스트로 시작하고 테스트로 끝난다.

```
버그 발견
  → 버그를 재현하는 실패 테스트 작성
  → 실패 확인
  → 디버깅 프로세스 진행
  → 수정
  → 테스트 통과 확인
  → 회귀 방지 테스트로 영구 보존
```

### Red Flags — 자기 검열 (기존 유지)

다음 생각이 들면 멈추고 프로세스를 따른다:

| 생각 | 현실 |
|------|------|
| "아마 이게 원인일 거야" | 증거 없는 추측. 조사부터 해라. |
| "일단 고쳐보고 되나 보자" | 추측 수정은 새 버그를 만든다. |
| "이건 간단한 문제야" | 간단해 보이는 버그가 가장 위험하다. |
| "이전에 비슷한 거 고쳤으니까" | 비슷하게 보여도 원인이 다를 수 있다. |
| "로그 한 줄만 추가하면 되겠지" | 로그 추가도 가설에 기반해야 한다. |

### 기존 대비 변경 요약

| 항목 | 기존 (superpowers) | 커스텀 |
|------|-------------------|--------|
| 유저 관여 시점 | 3회 실패 후 | 조사 완료 직후 (Standard), 즉시 (Complex) |
| 심각도 분기 | 없음 (모두 동일 프로세스) | Trivial / Standard / Complex 3단계 |
| 보고 형식 | 미정의 | 표준 보고 템플릿 |
| 에스컬레이션 | 3회 실패 → 유저 상의 | 3회 실패 → 설계 문제 의심 + 선택지 제시 |
| TDD 연계 | 언급만 | 실패 테스트 작성 필수, 회귀 테스트 보존 |
| 능동적 보고 | 유저 불만 시에만 | 프로세스에 보고 시점 내장 |

---

## Phase 5: 코드 리뷰

> 기반: `superpowers:requesting-code-review` (v5.0.7)
> 참고: everything-claude-code (멀티 에이전트 리뷰 파이프라인)

### 목적

구현된 코드를 **내부 병렬 리뷰 + Codex 외부 리뷰** 2단계로 검증하여 품질을 확보한다.

### 전체 흐름

```
① 내부 리뷰 (병렬 Agent Teams)
   → 코드 품질 / 보안 / 테스트 커버리지 동시 검토
   → 결과 종합

② 유저 확인
   → 리뷰 결과 요약 제시
   → Critical/Important 이슈 수정 방향 결정

③ 이슈 수정
   → 합의된 이슈 수정

④ Codex 외부 리뷰
   → 다른 관점에서 최종 검증

⑤ 최종 유저 확인
   → Codex 리뷰 결과 제시
   → 추가 수정 여부 결정
   → Phase 6 (검증)으로 전환
```

### 1단계: 내부 병렬 리뷰 (Agent Teams)

기존의 단일 code-reviewer 대신, **3개 에이전트를 병렬 실행**한다.

```
동시 실행:
  Agent A — 코드 품질 리뷰
    · 가독성, 네이밍
    · 설계 패턴, 구조
    · 이뮤터블 패턴 준수
    · 함수 크기 (<50줄), 파일 크기 (<800줄)
    · 중복 코드, 불필요한 복잡성

  Agent B — 보안 리뷰
    · 하드코딩된 시크릿
    · 입력 검증
    · SQL 인젝션, XSS 방지
    · 인증/인가 검증
    · 에러 메시지의 민감 정보 노출

  Agent C — 테스트 리뷰
    · 커버리지 80% 이상 확인
    · 테스트 품질 (mock 남용, 구현 의존성)
    · 엣지 케이스 누락
    · 테스트 격리성
```

각 에이전트에는 **세션 히스토리가 아닌 git diff + 스펙 문서만** 전달하여 객관적 평가를 보장한다.

### 리뷰 결과 종합

3개 에이전트의 결과를 종합하여 유저에게 제시한다.

```markdown
## 코드 리뷰 결과

### Critical (반드시 수정)
- [보안] src/auth/login.ts:23 — 비밀번호가 로그에 노출됨
- [품질] src/utils/parse.ts:45 — 입력 검증 누락

### Important (수정 추천)
- [품질] src/api/handler.ts:78 — 함수 68줄, 분리 권장
- [테스트] tests/auth.test.ts — 토큰 만료 엣지 케이스 미테스트

### Minor (유저 판단)
- [품질] src/types/index.ts:12 — 네이밍 개선 여지
```

### 유저 확인

```
리뷰 결과를 보고 유저가 결정:
  Critical → 반드시 수정 (유저 확인 후 진행)
  Important → 수정 추천, 유저가 선택
  Minor → 유저 판단에 맡김
```

### 2단계: Codex 외부 리뷰

내부 리뷰 이슈 수정 후, `codex-plugin-cc`를 통해 Codex에 리뷰를 요청한다.

```
config의 codex_review가 true일 때만 실행.
false면 이 단계를 건너뛰고 Phase 6으로 전환.
```

**호출:**

```
/codex:review                  → 일반 코드 리뷰
/codex:adversarial-review      → 더 강한 검증이 필요할 때
```

Codex는 uncommitted changes 또는 branch diff를 자동으로 읽어서 리뷰한다.

**Codex 리뷰 관점:**

```
- 스펙 대비 구현 일치 여부
- 내부 리뷰에서 놓친 이슈
- 코드 스멜, 성능 병목
- 아키텍처적 우려사항
- (adversarial) 설계 트레이드오프, 숨겨진 가정, 실패 모드
```

### Codex 리뷰 결과 처리

```
Codex 리뷰 결과 → 유저에게 제시
  → 동의하는 이슈 수정
  → 동의하지 않는 이슈는 유저 판단으로 스킵
  → 모든 처리 완료 후 Phase 6 (검증)으로 전환
```

### 기존 대비 변경 요약

| 항목 | 기존 (superpowers) | 커스텀 |
|------|-------------------|--------|
| 리뷰 구조 | 단일 code-reviewer 1회 | 내부 병렬 3팀 + Codex 외부 리뷰 |
| 리뷰 관점 | 통합 리뷰 | 품질 / 보안 / 테스트 분리 |
| 병렬 실행 | 없음 | Agent Teams 병렬 |
| 외부 리뷰 | 없음 | Codex 최종 리뷰 |
| 유저 관여 | 리뷰 결과 자동 반영 | 리뷰 결과마다 유저 확인 |
| 이슈 분류 | Critical/Important/Minor | 동일 (유지) + 유저 선택권 강화 |

---

## Phase 6: 검증

> 기반: `superpowers:verification-before-completion` (v5.0.7)
> 참고: claude-mem (검증 결과 세션 메모리 저장)

### 목적

코드 리뷰를 통과한 코드가 **실제로 동작하는지** 검증한다. "증거 없이 완료 주장 금지"가 철칙이다.

### 철칙

```
"should work", "I'm confident", "테스트가 통과할 것이다"
→ 전부 금지. 실행 결과를 보여줘야 한다.
```

### 전체 흐름

```
① 검증 항목 자동 감지
   → 프로젝트 설정 파일 기반으로 검증 커맨드 구성

② 검증 레벨 결정
   → 변경 규모에 따라 FULL / PARTIAL / QUICK

③ 검증 실행
   → IDENTIFY → RUN → READ → VERIFY → CLAIM

④ 결과 보고
   → 유저에게 검증 결과 요약 제시

⑤ 실패 시 대응
   → Trivial: 수정 제안 → Standard 이상: Phase 4 진입

⑥ 전체 통과 시
   → Phase 7 (마무리)로 전환
```

### 검증 항목 자동 감지 (기존 대비 추가)

Phase 진입 시 프로젝트 설정 파일을 읽어서 검증 항목을 자동 구성한다.

```
감지 소스         → 검증 커맨드

package.json      → npm test / npm run build / npm run lint / npx tsc --noEmit
pyproject.toml    → pytest / mypy / ruff check
Cargo.toml        → cargo test / cargo build / cargo clippy
go.mod            → go test ./... / go build ./... / go vet ./...
```

감지 실패 시 유저에게 직접 확인한다.

### 검증 레벨 (기존 대비 추가)

변경 규모에 따라 검증 범위를 조절한다.

```
FULL — 대규모 변경, 새 기능
  → 전체 테스트 + 빌드 + 린트 + 타입체크 + 커버리지 (80% 게이트)

PARTIAL — 중간 규모 변경
  → 관련 테스트 + 빌드 + 타입체크

QUICK — 소규모 변경 (설정, 문서 등)
  → 빌드 통과 확인만
```

### 5단계 게이트 (기존 유지)

각 검증 항목에 대해 5단계를 밟는다.

```
① IDENTIFY — 무엇을 검증해야 하는지 확인
② RUN      — 실제 커맨드 실행
③ READ     — 출력 결과 읽기
④ VERIFY   — 통과/실패 판단
⑤ CLAIM    — 증거 기반으로 결과 주장
```

CLAIM은 반드시 RUN의 실제 출력에 근거해야 한다. "실행하지 않았지만 통과할 것이다"는 금지.

### 결과 보고 (기존 대비 추가)

검증 완료 후 유저에게 요약을 제시한다.

```markdown
## 검증 결과

✅ 테스트: 42 passed, 0 failed (커버리지 87%)
✅ 빌드: 성공
✅ 린트: 경고 0
✅ 타입체크: 에러 0

전체 검증 통과. Phase 7 (마무리)로 진행할까요?
```

### 검증 실패 시

```
실패 항목 발견
  → 유저에게 보고 (어떤 항목이, 왜 실패했는지)
  → Trivial (린트 경고 등): 수정 제안 → 유저 승인 → 적용 → 재검증
  → Standard 이상: Phase 4 (디버깅) 진입
  → 수정 완료 후 전체 재검증
```

### 자기합리화 방지 (기존 유지)

| 생각 | 현실 |
|------|------|
| "테스트가 통과할 거야" | 실행해서 보여줘라. |
| "방금 고쳤으니까 될 거야" | 실행해서 보여줘라. |
| "사소한 변경이라 검증 불필요" | QUICK 레벨이라도 빌드는 확인해라. |
| "이전 검증에서 통과했으니까" | 그 이후 코드가 바뀌었으면 다시 실행해라. |
| "수동으로 확인했다" | 자동화된 검증 결과를 보여줘라. |

### 기존 대비 변경 요약

| 항목 | 기존 (superpowers) | 커스텀 |
|------|-------------------|--------|
| 검증 항목 | 일반 체크리스트 | 프로젝트 자동 감지 |
| 검증 레벨 | 모든 변경 동일 | FULL / PARTIAL / QUICK 분기 |
| 결과 보고 | 형식 없음 | 구조화된 요약 + 유저 제시 |
| 결과 기록 | 없음 | 검증 결과 로깅 |
| 실패 대응 | 명시적 흐름 없음 | 심각도별 대응 + Phase 4 연계 |

---

## Phase 7: 마무리

> 기반: `superpowers:finishing-a-development-branch` (v5.0.7)
> 참고: claude-mem (세션 종료 시 작업 내역 자동 캡처)

### 목적

검증을 통과한 코드를 **커밋하고 통합하는 마지막 단계**. 모든 행위는 유저 확인 후 진행한다.

### 전체 흐름

```
① 작업 완료 요약 제시
   → 전체 변경 내역, 커버리지, 리뷰 결과 종합

② 커밋
   → diff 요약 제시 → 유저 승인 → 커밋
   → 자동 커밋 없음

③ 통합 방식 선택
   → 유저가 4가지 중 선택

④ 선택에 따라 실행
   → 유저 확인 후 진행

⑤ 정리
   → worktree 정리 (해당 시)
   → /clear 권유
```

### 작업 완료 요약

마무리 진입 시 전체 작업 내역을 유저에게 제시한다.

```markdown
## 작업 완료 요약

**변경 내역:**
- 변경 파일: 8개 (신규 5, 수정 3)
- 삭제 파일: 0개

**품질:**
- 테스트 커버리지: 87%
- 리뷰 이슈: Critical 0, Important 2 (모두 수정 완료)
- 검증: 전체 통과 (테스트 ✅ 빌드 ✅ 린트 ✅ 타입체크 ✅)

**커밋 이력:**
- feat: add user authentication API
- feat: add JWT token generation
- test: add auth integration tests
- ...
```

### 커밋 — 항상 유저 확인

```
커밋 전:
  → git diff --stat 요약 제시
  → 커밋 메시지 초안 제시
  → 유저가 승인하면 커밋
  → 유저가 수정 요청하면 반영 후 재확인

커밋 메시지 형식:
  <type>: <description>
  (feat, fix, refactor, docs, test, chore, perf, ci)
```

### 통합 방식 선택

커밋 후 유저에게 4가지 선택지를 제시한다.

```
"커밋 완료. 어떻게 진행할까요?"

  (1) 로컬 머지 — 현재 브랜치를 base branch에 머지
  (2) PR 생성 — GitHub에 Pull Request 생성
  (3) 브랜치 유지 — 커밋만 하고 브랜치 그대로 유지
  (4) 폐기 — 브랜치와 변경사항 폐기
```

#### Option 1: 로컬 머지

```
① base branch 감지 (main → master → develop → 유저 확인)
② 머지 시도
③ 충돌 발생 시:
   → 충돌 파일 목록과 내용 유저에게 보고
   → 해결 방안 제시
   → 유저 확인 후 해결
   → 재머지
④ 머지 성공 → worktree 정리
```

#### Option 2: PR 생성

```
① base branch 감지
② 리모트에 push (유저 확인 후)
③ PR 생성
   → 제목: 70자 이내
   → 본문: 작업 완료 요약 기반으로 자동 작성
   → 테스트 계획 포함
④ PR URL 유저에게 전달
⑤ worktree 유지 (PR 머지 전까지)
```

#### Option 3: 브랜치 유지

```
① 추가 작업 여부 확인
② worktree 유지
③ 다음 세션에서 이어서 작업 가능
```

#### Option 4: 폐기

```
① 안전장치: "discard"를 정확히 입력해야 진행
② 브랜치 삭제
③ worktree 정리
④ 유저에게 폐기 완료 알림
```

### 정리

마무리 완료 후:

```
→ /clear 권유 (세션 짧게 유지 원칙)
→ 다음 작업이 있으면 새 세션에서 Phase 1부터 시작
```

### 기존 대비 변경 요약

| 항목 | 기존 (superpowers) | 커스텀 |
|------|-------------------|--------|
| 커밋 | 자동 실행 가능 | 항상 유저 확인 필수 |
| 작업 요약 | 없음 | 전체 작업 내역 요약 제시 |
| 머지 충돌 | 처리 미정의 | 충돌 보고 → 유저와 해결 |
| base branch | main/master만 | main → master → develop → 유저 확인 |
| 마무리 후 | 없음 | /clear 권유 |

---

## Workflow Config

### 목적

워크플로우에서 사용하는 경로, 설정값, 기본값들을 하나의 config 파일로 관리한다. 프로젝트마다 다른 설정을 적용할 수 있다.

### 파일 위치

```
.claude/workflow-config.yaml
```

### 구조

```yaml
# 산출물 저장 경로
paths:
  specs: docs/specs              # Phase 1 스펙 문서
  plans: docs/plans              # Phase 2 계획 문서
  workflows: .claude/workflows   # status + log 저장 디렉토리
  archive: .claude/workflow-archive

# 유저 관여 수준
engagement_level: balanced       # high / balanced / low
  # high:     모든 곳에서 유저 확인 (처음 익힐 때, 중요한 프로젝트)
  # balanced: Phase 전환 + 리뷰 결과 + 커밋 시에만 확인 (기본값)
  # low:      Phase 전환 시에만 확인 (신뢰도 높아진 후)

# 기본값
defaults:
  scale: auto                    # auto / small / standard / large
  checkpoint_interval: phase     # task / phase / end
  coverage_threshold: 80         # 커버리지 게이트 (%)
  review_mode: parallel          # parallel / single
  codex_review: true             # Codex 외부 리뷰 활성화

# 커밋
git:
  base_branches: [main, master, develop]
  message_format: conventional   # conventional commits

# TDD
tdd:
  framework: auto                # auto / jest / vitest / pytest / cargo / go
  guard_hook: true               # tdd-guard 훅 활성화
```

#### engagement_level 상세

```
high:
  Phase 전환 시 확인, 매 커밋 확인, 리뷰 결과 확인,
  디버깅 보고 확인, 검증 결과 확인
  → 확인 횟수: ~10회 (Standard 기준)

balanced (기본값):
  Phase 전환 시 확인, 리뷰 결과 확인, 최종 커밋 확인
  Phase 내부 세부 작업(TDD 사이클, 디버깅 조사 등)은 자동 진행
  이슈 발생 시에만 중간 보고
  → 확인 횟수: ~4회 (Standard 기준)

low:
  Phase 전환 시에만 확인
  나머지 전부 자동 진행
  → 확인 횟수: ~2회 (Standard 기준)
```

#### Small 규모 전 Phase 경량화

scale이 Small일 때 각 Phase의 동작:

```
Phase 1: 질문 1-2개 → 설계 요약 → 승인
Phase 2: 생략 (status에 간단 기록만)
Phase 3: TDD 동일 (코드 품질은 규모와 무관)
Phase 4: 필요 시에만
Phase 5: 단일 code-reviewer 1회 (review_mode 무시, Codex 생략)
Phase 6: QUICK 검증 (빌드 통과만)
Phase 7: 커밋 확인만
```

### 사용 규칙

```
모든 Phase 공통 — 시작 시:
  1. .claude/workflow-config.yaml 읽기
  2. .claude/workflows/ 에서 현재 워크플로우 status 파일 읽기 (있으면)
  3. 현재 작업에 사용 가능한 스킬 목록 확인 → 관련 스킬이 있으면 사용
  4. config 값에 따라 Phase 진행
```

이 절차는 **각 Phase 스킬의 첫 번째 단계로 명시**한다. superpowers 플러그인 없이도 독립적으로 작동해야 하므로, 외부 스킬(using-superpowers 등)에 의존하지 않는다.

config 파일이 없으면 위 기본값으로 동작한다. 프로젝트별로 필요한 값만 오버라이드하면 된다.

### 예시: 프로젝트별 커스텀

```yaml
# 프로젝트 A: 스펙 경로만 변경
paths:
  specs: docs/design

# 프로젝트 B: 커버리지 높이고 Codex 리뷰 비활성화
defaults:
  coverage_threshold: 90
  codex_review: false

# 프로젝트 C: Python 프로젝트, 단일 리뷰
tdd:
  framework: pytest
defaults:
  review_mode: single
```

---

## Workflow Status 관리

### 목적

/clear로 세션을 짧게 유지하면서도, 워크플로우 진행 상태를 잃지 않고 이어서 작업할 수 있게 한다.

### 파일 구조 (멀티 워크플로우 지원)

```
.claude/
  workflows/
    feat-payment.yaml          ← A 기능 상태
    feat-payment-log.md        ← A 기능 이력
    fix-auth-bug.yaml          ← B 핫픽스 상태
    fix-auth-bug-log.md        ← B 핫픽스 이력
  workflow-config.yaml          ← 공통 설정 (하나)
  workflow-archive/             ← 완료된 워크플로우 보관
```

파일명은 브랜치명 또는 프로젝트명 기반으로 자동 생성한다. 동시에 여러 워크플로우를 독립적으로 추적할 수 있다.

### workflow-status.yaml — 현재 상태

```yaml
project: 결제 시스템
branch: feat/payment-system
started: 2026-04-01
scale: standard  # small / standard / large

current:
  phase: 3
  phase_name: TDD
  task: 5
  total_tasks: 8
  coverage: 82%
  checkpoint_interval: phase  # task / phase / end
  last_updated: 2026-04-01T15:30:00

artifacts:
  spec: docs/specs/2026-04-01-payment-design.md
  plan: docs/plans/2026-04-01-payment.md

phases:
  1: { status: completed, summary: "JWT 기반 인증 + Stripe 연동" }
  2: { status: completed, summary: "8태스크, 병렬 그룹 2개" }
  3: { status: in_progress, summary: "Task 4/8 완료, 커버리지 82%" }
  4: { status: pending }
  5: { status: pending }
  6: { status: pending }
  7: { status: pending }

tasks:
  1: { name: "DB 스키마", status: completed }
  2: { name: "User 모델", status: completed }
  3: { name: "Auth 서비스", status: completed }
  4: { name: "Auth API", status: completed }
  5: { name: "미들웨어", status: in_progress }
  6: { name: "권한 관리", status: pending }
  7: { name: "에러 처리", status: pending }
  8: { name: "통합 테스트", status: pending }

notes:
  - "Task 3: bcrypt 5.1.0으로 고정 (하위 버전 호환 이슈)"
  - "Phase 2에서 체크포인트 간격: Phase 경계마다로 결정"
```

### workflow-log.md — append-only 이력

```markdown
# Workflow Log: 결제 시스템

## 2026-04-01 14:00 — Phase 1 완료
- 규모: Standard
- 접근법: JWT + Stripe (Redis Pub/Sub 대안 검토 후 결정)
- 스펙: docs/specs/2026-04-01-payment-design.md

## 2026-04-01 14:30 — Phase 2 완료
- 태스크 8개, 병렬 그룹: Task 1-2, Task 4-5
- 체크포인트: Phase 경계마다
- 계획: docs/plans/2026-04-01-payment.md

## 2026-04-01 15:00 — Task 3 이슈
- bcrypt 하위 버전 호환 이슈 발견 → 5.1.0 고정으로 해결

## 2026-04-01 15:30 — Task 4 완료
- 커버리지: 82%
- 커밋: feat: add auth API endpoint
```

### 업데이트 타이밍

| 이벤트 | status.yaml | log.md |
|--------|-------------|--------|
| Phase 전환 | phase, phase_name 업데이트 | Phase 완료 요약 append |
| 태스크 완료 | task, tasks[N].status 업데이트 | - |
| 커밋 | last_updated 갱신 | 커밋 메시지 append |
| 에러/이슈 발견 | notes에 추가 | 이슈 내용 append |
| 유저 결정사항 | 관련 필드 업데이트 | 결정 내용 append |
| 커버리지 변경 | coverage 업데이트 | - |

> 현재는 워크플로우 내에서 수동 업데이트. 추후 훅(PostToolUse, Stop)으로 자동화 가능.

### 세션 재개 흐름

/clear 후 새 세션에서:

```
① .claude/workflows/ 디렉토리 확인
② 진행 중인 워크플로우가 있으면 목록 제시:

  "진행 중인 워크플로우가 있습니다:
   (1) feat-payment — Phase 3 TDD, Task 5 (미들웨어)
   (2) fix-auth-bug — Phase 4 디버깅

   어떤 걸 이어서 진행할까요?
   (A) 번호 선택하여 이어서 진행
   (B) 특정 Phase/Task로 이동
   (C) 새 워크플로우 시작"

③ 유저 선택에 따라 진행
```

> 추후 SessionStart 훅으로 자동 주입 가능.

### 워크플로우 완료 시

```
Phase 7 완료
  → status yaml 전체 Phase ✅ 업데이트
  → log에 최종 완료 요약 append
  → 회고 진행
  → .claude/workflow-archive/YYYY-MM-DD-<project>.yaml 로 이동
  → .claude/workflow-archive/YYYY-MM-DD-<project>-log.md 로 이동
  → 다음 /vibe 시 깨끗한 상태에서 시작
```

### 회고 (Retrospective)

워크플로우 완료 후, 프로세스 자체를 개선하기 위한 간단한 피드백을 수집한다.

```
Phase 7 마무리 후:

  "이번 워크플로우에서:
   - 불필요하거나 과했던 단계가 있었나요?
   - 부족했던 부분은요?"

  유저 답변 예시:
    "Phase 5 리뷰가 과했다"
    "Phase 1 질문이 너무 많았다"

  → workflow-log.md에 회고 내용 append
  → 필요 시 workflow-config.yaml에 반영
    (예: review_mode: single, engagement_level: low 등)
```

이를 통해 워크플로우가 고정된 틀이 아니라, **쓸수록 자기에게 맞게 최적화되는 구조**가 된다.

> Small 프로젝트 전체 경로는 Workflow Config 섹션의 "Small 규모 전 Phase 경량화" 참조.

### Codex 리뷰 연동 — codex-plugin-cc

OpenAI 공식 Claude Code 플러그인 (`openai/codex-plugin-cc`)을 사용한다.

**설치:**

```
/plugin marketplace add openai/codex-plugin-cc
/plugin install codex@openai-codex
/codex:setup
```

요구사항: ChatGPT 구독(무료 포함) 또는 OpenAI API 키, Node.js 18.18+

**사용 가능한 커맨드:**

| 커맨드 | 용도 |
|--------|------|
| `/codex:review` | 코드 리뷰 (읽기 전용, 변경 없음) |
| `/codex:adversarial-review` | 공격적 리뷰 (설계 트레이드오프, 취약점 도전) |
| `/codex:rescue` | 태스크 위임 (Codex 서브에이전트로 작업 넘기기) |
| `/codex:status` | 백그라운드 작업 상태 확인 |
| `/codex:result` | 결과 확인 |
| `/codex:cancel` | 작업 취소 |

**Phase 5에서의 호출 흐름:**

```
① 내부 병렬 리뷰 완료 + 이슈 수정
② /codex:review 호출 (기본)
   또는 /codex:adversarial-review 호출 (더 강한 검증이 필요할 때)
③ Codex 리뷰 결과 수신
④ 결과 종합 → 유저 확인
```

**config 연동:**

```yaml
# workflow-config.yaml
defaults:
  codex_review: true     # false면 Codex 리뷰 생략, 내부 리뷰만으로 Phase 5 완료
```
