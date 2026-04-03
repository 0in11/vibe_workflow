# Vibe Workflow

AI Full-stack Engineer를 위한 커스텀 바이브코딩 워크플로우 플러그인.

브레인스토밍부터 코드 리뷰, 검증, 배포까지 — 하나의 커맨드로 전체 개발 파이프라인을 오케스트레이션합니다.

```
/vibe:feature "소셜 로그인 추가"
```

---

## Why

AI 코딩 도구를 쓰면서 이런 경험 있지 않나요?

- Claude가 설계 없이 바로 코딩부터 시작해서 꼬였다
- 테스트를 건너뛰고 "나중에 쓸게"라고 했다가 버그 터졌다
- /clear 했더니 이전 맥락을 다 잊었다
- 코드 리뷰를 아예 안 하거나 형식적으로만 했다
- 커밋을 물어보지도 않고 자동으로 해버렸다

Vibe Workflow는 이 문제들을 **구조적으로 해결**합니다.

---

## Features

### 3가지 모드

| 모드 | 커맨드 | 경로 | 용도 |
|------|--------|------|------|
| **Feature** | `/vibe:feature` | Phase 1→2→3→(4)→5→6→7 | 새 기능 구현 |
| **Hotfix** | `/vibe:hotfix` | Phase 4→3→6→7 | 긴급 버그 수정 |
| **Refactor** | `/vibe:refactor` | Phase 1→3→5→6→7 | 코드 구조 개선 |

### 7 Phase 파이프라인

```
Phase 1  브레인스토밍    아이디어 → 검증된 설계
Phase 2  계획 수립       설계 → 실행 가능한 태스크
Phase 3  TDD            RED → GREEN → REFACTOR → COVERAGE
Phase 4  디버깅          증거 기반 근본 원인 분석
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

### 유저 관여도 조절

`engagement_level` 하나로 확인 빈도를 제어합니다.

```yaml
# workflow-config.yaml
engagement_level: balanced    # high / balanced / low
```

| 레벨 | 확인 횟수 | 적합한 상황 |
|------|----------|------------|
| `high` | ~10회 | 처음 익힐 때, 중요한 프로젝트 |
| `balanced` | ~4회 | 일상 개발 (기본값) |
| `low` | ~2회 | 신뢰도 높아진 후, 반복 작업 |

### 세션 연속성

`/clear` 해도 맥락을 잃지 않습니다.

```
.claude/workflows/
  feat-payment.yaml        ← 상태 추적
  feat-payment-log.md      ← 이력 기록
```

새 세션에서 자동으로 이전 워크플로우를 감지하고 이어서 진행합니다.

---

## Install

```bash
# 마켓플레이스 등록
/plugin marketplace add 0in11/vibe_workflow

# 플러그인 설치
/plugin install vibe@0in11-vibe-workflow
```

### 선택사항: Codex 외부 리뷰

Phase 5에서 Codex 코드 리뷰를 사용하려면:

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
# 산출물 경로
paths:
  specs: docs/specs
  plans: docs/plans
  workflows: .claude/workflows
  archive: .claude/workflow-archive

# 유저 관여 수준
engagement_level: balanced     # high / balanced / low

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
  docs/
    design.md               # 설계 문서
```

---

## Design Principles

- **주요 결정은 유저가 한다** — Claude는 분석/제안, 유저가 선택
- **증거 기반** — 추측 금지, 서치와 실행 결과로 판단
- **TDD 강제** — 테스트 없는 코드는 삭제 (Iron Law)
- **세션 짧게** — /clear 자주, status 파일로 연속성 유지
- **스킬 적극 활용** — 관련 스킬이 있으면 자동으로 사용
- **회고** — 워크플로우 완료 후 프로세스 자체를 개선

---

## Benchmarks

기존 superpowers 스킬셋과 커뮤니티 도구들을 벤치마킹하여 개선했습니다.

| 기반 | 가져온 것 | 개선한 것 |
|------|----------|----------|
| superpowers:brainstorming | Hard Gate, 질문 패턴 | 규모 분기, 근거 기반 서치, MVP 구분 |
| superpowers:writing-plans | 플레이스홀더 금지, 태스크 분해 | 유저 확인 게이트, 의존성 그래프, 테스트 명세 |
| superpowers:tdd | RED-GREEN-REFACTOR, 합리화 방어 | 커버리지 게이트, Spike 경로, 프레임워크 자동 감지 |
| superpowers:systematic-debugging | 증거 기반 디버깅 | 심각도 분기, 유저 보고 앞당김, 보고 템플릿 |
| tdd-guard | 훅 기반 TDD 강제 | 스킬 + 훅 이중 안전망 |
| codex-plugin-cc | Codex 리뷰 | 내부 병렬 리뷰 + Codex 2단계 구조 |

---

## License

MIT
