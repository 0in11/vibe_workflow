---
name: vibe:hotfix
description: "프로덕션 장애 등 긴급 수정이 필요할 때 사용하는 핫픽스 워크플로우. 디버깅 직행→수정→검증→마무리 최단 경로."
---

# Vibe Hotfix Workflow

긴급 수정을 위한 최단 경로: Phase 4 → Phase 3(수정) → Phase 6(QUICK) → Phase 7

## Phase 시작 공통 절차

**모든 Phase 진입 시 반드시 실행:**

1. `.claude/workflow-config.yaml` 읽기 (없으면 기본값 사용)
2. `.claude/workflows/` 에서 현재 워크플로우 status 파일 읽기 (있으면)
3. 사용 가능한 스킬 목록을 조회하고, 현재 Phase의 작업과 1%라도 관련된 스킬이 있으면 반드시 호출한다. 스킬을 쓸지 말지 고민되면, 쓴다.
4. config 값에 따라 Phase 진행

## engagement_level

config의 `engagement_level`에 따라 유저 확인 빈도를 조절한다.
핫픽스는 긴급하므로, balanced/low에서는 최소한의 확인만 한다.

- **high**: 디버깅 보고 + 수정 방향 + 커밋 확인
- **balanced**: 디버깅 보고 + 커밋 확인
- **low**: 커밋 확인만

---

## Phase 4: 디버깅 직행

핫픽스이므로 심각도 판단 없이 **Standard 프로세스로 즉시 진입**한다.

```
증거 없이 수정 금지. "아마 이것 때문일 것이다" → 금지. "일단 고쳐보자" → 금지.
```

### 4.1 원인 조사

1. **에러 메시지, 스택 트레이스 수집** — 로그, 에러 리포트, 재현 조건 파악
2. **관련 코드 분석** — 에러 발생 지점, 최근 변경사항, 영향 범위
3. **근본 원인 후보 도출** — 증거 기반으로 1-3개 후보

### 4.2 유저에게 보고

```markdown
## 핫픽스 디버깅 보고

**증상:** [관찰된 현상 — 에러 메시지, 예상과 다른 동작]

**조사 범위:** [확인한 파일, 로그, 컴포넌트]

**근본 원인 후보:**
1. [후보 A] — 확신도: 높음 — 증거: [구체적 증거]
2. [후보 B] — 확신도: 중간 — 증거: [구체적 증거]

**추천 해결 방향:** [설명]

**유저 결정 필요:** [어떤 방향으로 갈지]
```

유저와 방향을 결정한 후 Phase 3으로 진행.

### 4.3 에스컬레이션

3회 수정 실패 시:
- "이 문제는 핫픽스로 해결하기 어려울 수 있습니다"
- 선택지: (A) 우회 방안 탐색 (B) /vibe:feature로 전환하여 본격 대응 (C) 계속 시도

### Status 업데이트

Phase 4 완료 시: status에 phase=4 completed, 근본 원인 요약 기록. log에 디버깅 보고 append.

---

## Phase 3: 수정

버그를 수정하고 테스트로 검증한다.

### 3.1 버그 재현 테스트 (필수)

1. 버그를 재현하는 실패하는 테스트 작성
2. 실행하여 실패 확인 (MANDATORY)
3. 실패 이유가 보고된 버그와 일치하는지 확인

### 3.2 수정

1. 최소한의 코드 변경으로 버그 수정
2. 재현 테스트 통과 확인 (MANDATORY)
3. 기존 테스트도 전부 통과 확인

### 3.3 회귀 테스트

작성한 재현 테스트는 회귀 방지 테스트로 영구 보존한다.

### Status 업데이트

수정 완료 시: status에 phase=3 completed, 수정 내용 기록. log에 수정 사항 append.

---

## Phase 6: QUICK 검증

핫픽스이므로 FULL 검증이 아닌 **QUICK 검증**만 수행한다.

```
"should work" → 금지. 실행 결과를 보여줘야 한다.
```

### 검증 항목

1. **빌드 통과** 확인
2. **관련 테스트 통과** 확인 (전체 테스트 불필요)
3. 5단계 게이트: IDENTIFY → RUN → READ → VERIFY → CLAIM

### 결과 보고

```markdown
## 검증 결과
✅ 빌드: 성공
✅ 관련 테스트: N passed, 0 failed
```

### 실패 시

실패하면 Phase 4로 돌아가서 추가 조사.

### Status 업데이트

Phase 6 완료 시: status에 phase=6 completed. log에 검증 결과 append.

---

## Phase 7: 마무리

긴급성을 반영하여 빠르게 커밋하고 통합한다.

### 7.1 커밋

- diff 요약 + 커밋 메시지 초안 제시 → 유저 승인 → 커밋
- 메시지 형식: `fix: <description>`

### 7.2 통합 방식

핫픽스이므로 즉시 통합을 기본으로 제안:

```
"핫픽스 커밋 완료. 어떻게 진행할까요?"

  (1) 즉시 머지 ← 긴급 대응 시 추천
  (2) PR 생성 — 리뷰가 필요한 경우
  (3) 브랜치 유지
```

base branch 감지: config의 `git.base_branches` 순서대로 시도.

### 7.3 회고

```
"이번 핫픽스에서:
 - 원인 파악에 걸린 시간이 적절했나요?
 - 예방할 수 있었던 문제였나요?"
```

→ log에 회고 append

### 7.4 정리

- status/log → `workflow-archive/`로 이동
- /clear 권유

---

## Status 관리

### 워크플로우 시작 시

`.claude/workflows/fix-<bug-name>.yaml` 과 `fix-<bug-name>-log.md` 생성.

### 업데이트 타이밍

| 이벤트 | status yaml | log md |
|--------|-------------|--------|
| Phase 전환 | phase 업데이트 | 완료 요약 append |
| 수정 완료 | task status 업데이트 | 수정 내용 append |
| 커밋 | last_updated 갱신 | 커밋 메시지 append |
| 에스컬레이션 | notes에 추가 | 상세 기록 append |

### 완료 시

`.claude/workflow-archive/YYYY-MM-DD-fix-<bug-name>.*` 로 이동.
