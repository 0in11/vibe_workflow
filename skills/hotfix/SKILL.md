---
name: vibe:hotfix
description: "프로덕션 장애 등 긴급 수정이 필요할 때 사용하는 핫픽스 워크플로우. 디버깅 직행 최단 경로."
---

# Vibe Hotfix Workflow

긴급 수정을 위한 최단 경로.

## Phase 시작 공통 절차

**모든 Phase 진입 시 반드시 실행:**

1. `.claude/workflow-config.yaml` 읽기 (없으면 기본값 사용)
2. `.claude/workflows/` 에서 현재 워크플로우 status 파일 읽기 (있으면)
3. 사용 가능한 스킬 목록을 조회하고, 현재 Phase의 작업과 1%라도 관련된 스킬이 있으면 반드시 호출한다. 스킬을 쓸지 말지 고민되면, 쓴다.
4. config 값에 따라 Phase 진행

## engagement_level

- **high**: 디버깅 보고 + 수정 방향 + 커밋 확인
- **balanced**: 디버깅 보고 + 커밋 확인
- **low**: 커밋 확인만

## 파이프라인

```
Phase 4 → Phase 3 → Phase 6 (QUICK) → Phase 7
```

### Phase 4: 디버깅 직행
**vibe:debugging** 스킬을 호출한다.
- 심각도 판단 없이 Standard 프로세스로 즉시 진입
- 원인 파악 → 유저 보고 → 방향 결정

### Phase 3: 수정
**vibe:tdd** 스킬을 호출한다.
- 버그 재현 테스트 작성 (필수)
- 수정 → 테스트 통과 확인
- 기존 테스트도 전부 통과 확인

### Phase 6: QUICK 검증
**vibe:verification** 스킬을 호출한다.
- QUICK 레벨: 빌드 + 관련 테스트 통과만

### Phase 7: 마무리
**vibe:finishing** 스킬을 호출한다.
- 커밋 확인 → 즉시 머지 권장 (긴급성)
- 회고: "원인 파악 시간이 적절했나? 예방 가능한 문제였나?"

## Status 관리

워크플로우 시작 시 `.claude/workflows/fix-<bug-name>.yaml` + log 생성.
완료 시 `workflow-archive/`로 이동.
