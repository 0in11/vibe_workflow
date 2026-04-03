# Vibe Workflow

AI 코딩, 흐름을 타면서도 품질을 놓치지 않는 워크플로우.

```
/vibe:feature "소셜 로그인 추가"
```

하나의 커맨드로 브레인스토밍부터 코드 리뷰, 검증, 배포까지.

---

## 이런 경험, 있지 않나요?

> "Claude가 물어보지도 않고 혼자 코드를 짜기 시작했는데, 전혀 다른 방향이었다"

> "테스트 나중에 쓴다더니 결국 안 썼다"

> "/clear 했더니 뭘 하고 있었는지 다 잊어버렸다"

> "접근법을 추천해줬는데 근거가 없었다. 나중에 알고보니 deprecated된 방식이었다"

> "커밋을 자동으로 해버려서 되돌려야 했다"

> "관련 스킬이 분명 있는데 안 쓰고 맨땅에 헤딩한다"

Vibe Workflow는 이 문제들을 하나하나 구조적으로 해결합니다.

---

## 핵심 기능

### AI가 추측하지 않는다

접근법을 제안할 때 **반드시 근거를 서치**합니다. 공식 문서(context7), 실제 사례(WebSearch), GitHub 구현체를 병렬로 조사한 후 제안합니다. "아마 이게 좋을 것 같다"는 없습니다.

```
재료는 검증된 것만, 요리(조합/판단)는 적극적으로.
```

### 결정은 항상 내가 한다

Claude는 분석하고 제안합니다. 선택은 유저가 합니다. 설계 승인, 구현 계획 확정, 커밋, 머지 — 모든 핵심 결정에 유저 확인이 있습니다. 그리고 그 빈도를 **내가 조절**할 수 있습니다.

```yaml
engagement_level: balanced    # high(~10회) / balanced(~4회) / low(~2회)
```

### 스킬을 적극적으로 쓴다

매 Phase 진입 시 사용 가능한 스킬을 조회하고, 1%라도 관련이 있으면 반드시 호출합니다. "스킬을 쓸지 말지 고민되면, 쓴다"가 원칙입니다.

### /clear 해도 이어서 진행

세션을 짧게 유지하면서도 맥락을 잃지 않습니다. 워크플로우 상태를 YAML로 추적하고, 이력을 로그로 남깁니다. 새 세션에서 자동으로 이전 작업을 감지합니다.

```
"진행 중인 워크플로우가 있습니다:
  (1) feat-payment — Phase 3 TDD, Task 5/8
  어떤 걸 이어서 진행할까요?"
```

### TDD를 물리적으로 강제

"나중에 쓸게"는 통하지 않습니다. 테스트 없이 작성된 코드는 삭제합니다(Iron Law). 12가지 합리화 변명을 사전에 차단하고, tdd-guard 훅이 2차 안전망으로 작동합니다.

```
RED → GREEN → REFACTOR → COVERAGE(80%)
```

### 코드 리뷰가 진짜로 돌아간다

형식적 리뷰가 아닙니다. 내부에서 품질/보안/테스트 3개 에이전트가 병렬로 리뷰하고, Codex가 외부 리뷰어로 한 번 더 검증합니다.

```
내부: Agent 3팀 병렬 (품질 / 보안 / 테스트)
외부: /codex:review 또는 /codex:adversarial-review
```

### 에러가 나면 같이 해결한다

Claude가 혼자 3번 시도하고 실패한 후에야 알려주는 게 아닙니다. 원인 분석은 자동으로 하되, **해결 방향은 유저와 함께** 결정합니다. 심각도에 따라 보고 시점도 다릅니다.

```
Trivial:  즉시 수정 제안
Standard: 조사 후 보고 → 같이 방향 결정
Complex:  즉시 보고 → 전 과정 협업
```

### 쓸수록 나한테 맞아간다

워크플로우 완료 후 회고를 합니다. "이번에 뭐가 과했고, 뭐가 부족했는지" 피드백하면 config에 반영됩니다. 고정된 틀이 아니라, 쓸수록 최적화되는 구조입니다.

---

## 3가지 모드

| 모드 | 커맨드 | 경로 | 용도 |
|------|--------|------|------|
| **Feature** | `/vibe:feature` | Phase 1→2→3→(4)→5→6→7 | 새 기능 구현 |
| **Hotfix** | `/vibe:hotfix` | Phase 4→3→6→7 | 긴급 버그 수정 |
| **Refactor** | `/vibe:refactor` | Phase 1→3→5→6→7 | 코드 구조 개선 |

### 7 Phase 파이프라인

```
Phase 1  브레인스토밍    아이디어 → 검증된 설계 (근거 기반 서치)
Phase 2  계획 수립       설계 → 실행 가능한 태스크 (의존성 그래프)
Phase 3  TDD            RED → GREEN → REFACTOR → COVERAGE
Phase 4  디버깅          증거 기반 근본 원인 분석 (심각도 분기)
Phase 5  코드 리뷰       내부 병렬 3팀 + Codex 외부 리뷰
Phase 6  검증            빌드/테스트/린트 실제 실행 확인
Phase 7  마무리          커밋 → PR/머지 → 회고
```

### 규모 적응형

프로젝트 크기에 따라 프로세스가 자동 조절됩니다.

```
Small    설정 파일 한 줄 수정     →  35분  (축약 프로세스)
Standard 소셜 로그인 추가         →  90분  (전체 프로세스)
Large    결제 시스템 구축         →  단계별 진행
```

Small은 Phase 2 생략, 단일 리뷰어, QUICK 검증으로 빠르게 끝납니다.

---

## Install

```bash
# 마켓플레이스 등록
/plugin marketplace add 0in11/vibe_workflow

# 플러그인 설치
/plugin install vibe@0in11-vibe-workflow
```

### 선택사항: Codex 외부 리뷰

```bash
/plugin marketplace add openai/codex-plugin-cc
/plugin install codex@openai-codex
/codex:setup
```

---

## Quick Start

```bash
# 새 기능 구현
/vibe:feature "사용자 인증 API 추가"

# 긴급 버그 수정
/vibe:hotfix "로그인 시 500 에러"

# 코드 리팩터링
/vibe:refactor "auth 모듈 책임 분리"
```

---

## Configuration

프로젝트 루트에 `.claude/workflow-config.yaml`을 생성하면 프로젝트별 설정이 가능합니다.

```yaml
# 유저 관여 수준
engagement_level: balanced     # high / balanced / low

# 산출물 경로
paths:
  specs: docs/specs
  plans: docs/plans
  workflows: .claude/workflows
  archive: .claude/workflow-archive

# 기본값
defaults:
  scale: auto                  # auto / small / standard / large
  checkpoint_interval: phase   # task / phase / end
  coverage_threshold: 80       # 커버리지 게이트 (%)
  review_mode: parallel        # parallel / single
  codex_review: true           # Codex 외부 리뷰

# TDD
tdd:
  framework: auto              # auto / jest / vitest / pytest / cargo / go
  guard_hook: true             # tdd-guard 훅

# Git
git:
  base_branches: [main, master, develop]
  message_format: conventional
```

config 파일이 없으면 위 기본값으로 동작합니다.

---

## Architecture

```
vibe_workflow/
  .claude-plugin/
    plugin.json             # 플러그인 매니페스트
  skills/
    feature/SKILL.md        # /vibe:feature
    hotfix/SKILL.md         # /vibe:hotfix
    refactor/SKILL.md       # /vibe:refactor
  workflow-config.yaml      # 기본 설정 템플릿
```

---

## Benchmarks

기존 도구들을 벤치마킹하고 개선했습니다.

| 기반 | 가져온 것 | 개선한 것 |
|------|----------|----------|
| superpowers:brainstorming | Hard Gate, 질문 패턴 | 규모 분기, 근거 기반 서치, MVP 구분 |
| superpowers:writing-plans | 플레이스홀더 금지, 태스크 분해 | 유저 확인 게이트, 의존성 그래프, 테스트 명세 |
| superpowers:tdd | RED-GREEN-REFACTOR | 커버리지 게이트, Spike 경로, 프레임워크 자동 감지 |
| superpowers:systematic-debugging | 증거 기반 디버깅 | 심각도 분기, 유저 보고 앞당김 |
| tdd-guard | 훅 기반 TDD 강제 | 스킬 + 훅 이중 안전망 |
| codex-plugin-cc | Codex 리뷰 | 내부 병렬 + Codex 2단계 구조 |

---

## License

MIT
