---
name: vibe:finishing
description: "검증을 통과한 코드를 커밋하고 통합하는 마지막 Phase. 회고 포함."
---

# Phase 7: 마무리

검증을 통과한 코드를 커밋하고 통합한다.

## 작업 완료 요약

```markdown
## 작업 완료 요약
**변경 내역:** 변경 파일 N개 (신규 N, 수정 N)
**품질:** 커버리지 N%, 리뷰 이슈 모두 해결, 검증 전체 통과
**커밋 이력:** [목록]
```

## 커밋

diff 요약 + 커밋 메시지 초안 제시 → 유저 승인 → 커밋. 자동 커밋 금지.
메시지 형식: `<type>: <description>` (config의 `git.message_format`).

## 통합 방식 선택

```
(1) 로컬 머지 — base branch에 머지 (충돌 시 유저와 해결)
(2) PR 생성 — GitHub PR (제목 70자 이내, 본문 자동)
(3) 브랜치 유지 — 커밋만, 브랜치 유지
(4) 폐기 — "discard" 입력 필요
```

base branch 감지: config의 `git.base_branches` 순서대로 시도.

## 회고

```
"이번 워크플로우에서:
 - 불필요하거나 과했던 단계가 있었나요?
 - 부족했던 부분은요?"
```

→ log에 회고 내용 append → 필요 시 config에 반영

## 정리

- status/log를 `workflow-archive/`로 이동
- /clear 권유

## Status 업데이트

Phase 7 완료 시: status 전체 Phase ✅. log에 최종 요약 + 회고 append. archive로 이동.
