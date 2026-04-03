---
name: vibe:refactor
description: "기능 변경 없이 코드 구조를 개선할 때 사용하는 리팩터링 워크플로우."
---

# Vibe Refactor Workflow

코드 구조 개선을 위한 리팩터링 경로.

> **핵심 원칙: 기존 테스트가 깨지지 않는다.**
> Iron Law("테스트 없는 코드 삭제")는 적용되지 않는다. 이미 테스트가 있는 기존 코드를 다루기 때문이다.

## Phase 시작 공통 절차

**모든 Phase 진입 시 반드시 실행:**

1. `.claude/workflow-config.yaml` 읽기 (없으면 기본값 사용)
2. `.claude/workflows/` 에서 현재 워크플로우 status 파일 읽기 (있으면)
3. 사용 가능한 스킬 목록을 조회하고, 현재 Phase의 작업과 1%라도 관련된 스킬이 있으면 반드시 호출한다. 스킬을 쓸지 말지 고민되면, 쓴다.
4. config 값에 따라 Phase 진행

## engagement_level

- **high**: 설계 승인 + 리팩터링 중간 확인 + 리뷰 결과 + 커밋 확인
- **balanced**: 설계 승인 + 리뷰 결과 + 커밋 확인
- **low**: 설계 승인 + 커밋 확인만

## 파이프라인

```
Phase 1 → Phase 3 → Phase 5 → Phase 6 → Phase 7
```

### Phase 1: 설계
**vibe:brainstorming** 스킬을 호출한다. 단, 리팩터링 모드:
- 리팩터링 범위와 목표 정의
- **기능 변경이 아닌 구조 변경임을 확인** — 기능 변경 포함 시 /vibe:feature 전환 권유

### Phase 3: 리팩터링
**vibe:tdd** 스킬을 호출한다. 단, 리팩터링 모드:
- Iron Law 적용 안 됨
- 순서: 기존 테스트 전부 통과 확인 → 리팩터링 → 테스트 여전히 통과 확인
- 필요 시 테스트 추가
- 커버리지가 리팩터링 전보다 떨어지면 안 됨

### Phase 5: 코드 리뷰
**vibe:code-review** 스킬을 호출한다.
- 리뷰 관점 추가: 기존 기능이 변경되지 않았는지 확인

### Phase 6: 검증
**vibe:verification** 스킬을 호출한다.
- 핵심: 모든 기존 테스트 통과 + 커버리지 유지

### Phase 7: 마무리
**vibe:finishing** 스킬을 호출한다.
- 커밋 메시지: `refactor: <description>`
- 회고: "범위가 적절했나? 테스트가 충분했나?"

## Status 관리

워크플로우 시작 시 `.claude/workflows/refactor-<target>.yaml` + log 생성.
완료 시 `workflow-archive/`로 이동.
