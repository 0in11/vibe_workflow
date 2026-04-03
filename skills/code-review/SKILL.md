---
name: vibe:code-review
description: "내부 병렬 리뷰(Subagent 3팀) + Codex 외부 리뷰 2단계로 코드 품질을 검증한다."
---

# Phase 5: 코드 리뷰

구현된 코드를 내부 병렬 리뷰 + Codex 외부 리뷰 2단계로 검증한다.

## 1단계: 내부 리뷰 (Subagent 병렬)

**Standard/Large (review_mode: parallel):** 3개 Subagent 병렬 디스패치

```
Subagent A → 코드 품질 (가독성, 패턴, 이뮤터블, 함수<50줄, 파일<800줄)
Subagent B → 보안 (시크릿, 입력 검증, SQL 인젝션, XSS, 인증/인가)
Subagent C → 테스트 (커버리지 80%+, mock 남용, 엣지 케이스, 격리성)

각 Subagent는 독립적으로 리뷰하고 결과만 반환.
메인 에이전트가 3개 결과를 종합.
```

> 리뷰는 독립적 관점이 중요하므로 에이전트 간 소통이 필요 없다. Subagent가 적합.

**Small 또는 review_mode: single:** 단일 code-reviewer Subagent.

각 Subagent에는 세션 히스토리가 아닌 **git diff + 스펙 문서만** 전달하여 객관적 평가를 보장.

## 리뷰 결과 종합

```markdown
## 코드 리뷰 결과
### Critical (반드시 수정)
- [보안] src/auth/login.ts:23 — 비밀번호가 로그에 노출됨
### Important (수정 추천)
- [품질] src/api/handler.ts:78 — 함수 68줄, 분리 권장
### Minor (유저 판단)
- [품질] src/types/index.ts:12 — 네이밍 개선 여지
```

유저 확인 → Critical 수정 필수, Important는 유저 선택, Minor는 유저 판단.

## 2단계: Codex 외부 리뷰

config의 `defaults.codex_review`가 true일 때만 실행. Small이면 생략.

```
/codex:review                → 일반 코드 리뷰
/codex:adversarial-review    → 더 강한 검증이 필요할 때
```

Codex 결과 → 유저에게 제시 → 동의하는 이슈 수정, 아닌 건 스킵 → vibe:verification 스킬로 전환.

## Status 업데이트

Phase 5 완료 시: status에 phase=5 completed. log에 리뷰 결과 요약 append.
