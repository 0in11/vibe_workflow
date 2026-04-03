---
name: vibe:feature
description: "새 기능 구현을 위한 전체 바이브코딩 워크플로우. 브레인스토밍→계획→TDD→리뷰→검증→마무리를 오케스트레이션한다."
---

# Vibe Feature Workflow

새 기능을 구현하는 전체 파이프라인.

## Phase 시작 공통 절차

**모든 Phase 진입 시 반드시 실행:**

1. `.claude/workflow-config.yaml` 읽기 (없으면 기본값 사용)
2. `.claude/workflows/` 에서 현재 워크플로우 status 파일 읽기 (있으면 이어서 진행)
3. 사용 가능한 스킬 목록을 조회하고, 현재 Phase의 작업과 1%라도 관련된 스킬이 있으면 반드시 호출한다. 스킬을 쓸지 말지 고민되면, 쓴다.
4. config 값에 따라 Phase 진행

**세션 재개 시:**
`.claude/workflows/` 에 진행 중인 워크플로우가 있으면 목록을 제시하고 유저에게 선택받는다.

## engagement_level

config의 `engagement_level`에 따라 유저 확인 빈도를 조절한다.

- **high**: 모든 Phase 전환, 매 커밋, 리뷰 결과, 디버깅 보고, 검증 결과 확인 (~10회)
- **balanced** (기본): Phase 전환, 리뷰 결과, 최종 커밋만 확인. Phase 내부는 자동 진행 (~4회)
- **low**: Phase 전환 시에만 확인 (~2회)

## 파이프라인

```
Phase 1 → Phase 2 → Phase 3 → (Phase 4) → Phase 5 → Phase 6 → Phase 7
```

### Phase 1: 브레인스토밍
**vibe:brainstorming** 스킬을 호출한다.
- Small로 판단되면 Phase 2를 건너뛰고 Phase 3으로 직행

### Phase 2: 계획 수립
**vibe:planning** 스킬을 호출한다.
- 태스크 분해, 의존성 그래프, 유저 확인 게이트
- 실행 핸드오프: Subagent 기반(병렬 가능 태스크 동시 디스패치) 또는 인라인 실행

### Phase 3: TDD
**vibe:tdd** 스킬을 호출한다.
- RED → GREEN → REFACTOR → COVERAGE
- Subagent로 병렬 가능 태스크 동시 실행
- 2회 이상 수정 실패 시 → Phase 4 자동 진입

### Phase 4: 디버깅 (필요 시에만)
**vibe:debugging** 스킬을 호출한다.
- Trivial/Standard: Subagent로 조사
- Complex: Agent Teams로 협업 디버깅 (에이전트 간 소통 필요)
- 해결 후 Phase 3으로 복귀

### Phase 5: 코드 리뷰
**vibe:code-review** 스킬을 호출한다.
- 내부: Subagent 3개 병렬 (품질/보안/테스트) — Small이면 단일
- 외부: /codex:review (config에서 활성화 시)

### Phase 6: 검증
**vibe:verification** 스킬을 호출한다.
- FULL/PARTIAL/QUICK 레벨

### Phase 7: 마무리
**vibe:finishing** 스킬을 호출한다.
- 커밋 (유저 확인) → 통합 방식 선택 → 회고 → /clear 권유

## Status 관리

워크플로우 시작 시 `.claude/workflows/<branch-name>.yaml` + `<branch-name>-log.md` 생성.
각 Phase 전환, 태스크 완료, 커밋, 이슈 발견 시 업데이트.
완료 시 `workflow-archive/`로 이동.
