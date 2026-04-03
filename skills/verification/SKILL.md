---
name: vibe:verification
description: "코드가 실제로 동작하는지 검증한다. 증거 없이 완료 주장 금지."
---

# Phase 6: 검증

코드 리뷰를 통과한 코드가 실제로 동작하는지 검증한다.

```
"should work", "I'm confident" → 금지. 실행 결과를 보여줘야 한다.
```

## 검증 항목 자동 감지

프로젝트 설정 파일로 검증 커맨드 자동 구성:

```
package.json → npm test / npm run build / npm run lint / npx tsc --noEmit
pyproject.toml → pytest / mypy / ruff check
Cargo.toml → cargo test / cargo build / cargo clippy
go.mod → go test ./... / go build ./... / go vet ./...
```

감지 실패 시 유저에게 확인.

## 검증 레벨

- **FULL** (Standard/Large): 전체 테스트 + 빌드 + 린트 + 타입체크 + 커버리지
- **PARTIAL** (중간 변경): 관련 테스트 + 빌드 + 타입체크
- **QUICK** (Small/설정 변경/Hotfix): 빌드 + 관련 테스트만

## 5단계 게이트

각 검증 항목에 대해:

```
① IDENTIFY — 무엇을 검증해야 하는지 확인
② RUN      — 실제 커맨드 실행
③ READ     — 출력 결과 읽기
④ VERIFY   — 통과/실패 판단
⑤ CLAIM    — 증거 기반으로 결과 주장
```

CLAIM은 반드시 RUN의 실제 출력에 근거.

## 결과 보고

```markdown
## 검증 결과
✅ 테스트: N passed, 0 failed (커버리지 N%)
✅ 빌드: 성공
✅ 린트: 경고 0
✅ 타입체크: 에러 0
```

## 실패 시

Trivial: 수정 제안 → 유저 승인 → 재검증. Standard 이상: vibe:debugging 스킬 호출.

## 자기합리화 방지

| 생각 | 현실 |
|------|------|
| "테스트가 통과할 거야" | 실행해서 보여줘라. |
| "방금 고쳤으니까 될 거야" | 실행해서 보여줘라. |
| "사소한 변경이라 검증 불필요" | QUICK이라도 빌드는 확인해라. |
| "이전 검증에서 통과했으니까" | 코드가 바뀌었으면 다시 실행해라. |

## Status 업데이트

Phase 6 완료 시: status에 phase=6 completed, 검증 결과 기록. log에 결과 append.
