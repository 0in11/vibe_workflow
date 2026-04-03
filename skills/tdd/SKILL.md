---
name: vibe:tdd
description: "테스트 명세를 실제 코드로 변환하며 RED→GREEN→REFACTOR→COVERAGE 사이클로 구현한다."
---

# Phase 3: TDD

Phase 2 테스트 명세를 실제 코드로 변환하며 RED→GREEN→REFACTOR 사이클로 구현한다.

## Iron Law

```
테스트 없이 작성된 프로덕션 코드는 삭제한다.
참고용으로 보지 않고, 적응시키지 않고, 삭제한다.
```

## 프레임워크 자동 감지

Phase 3 진입 시 프로젝트 설정 파일로 자동 감지. config의 `tdd.framework`이 auto가 아니면 그 값 사용.

```
package.json (jest) → npx jest / npx jest --coverage
package.json (vitest) → npx vitest run / npx vitest run --coverage
pyproject.toml → pytest / pytest --cov
Cargo.toml → cargo test / cargo tarpaulin
go.mod → go test ./... / go test -cover ./...
```

## TDD 사이클

**RED** — 테스트 명세 기반으로 실패하는 테스트 작성. 실행하여 실패 확인 (MANDATORY).
**GREEN** — 테스트를 통과시키는 가장 단순한 코드. 실행하여 통과 확인 (MANDATORY).
**REFACTOR** — GREEN 이후에만. 체크: 긴 함수(>50줄), 깊은 중첩(>4레벨), 뮤테이션, 매직 넘버, 3회 이상 중복. 리팩터링 후 테스트 재실행 필수.
**COVERAGE** — config의 `defaults.coverage_threshold` 이상인지 확인. 미만이면 추가 TDD 사이클.

## Spike — 탐색적 코딩

불확실한 기술일 때:
1. Spike 선언 (유저에게 알림)
2. 자유롭게 구현 (TDD 일시 해제)
3. 결과 정리
4. Spike 코드 삭제
5. TDD로 재구현

> Spike는 예외가 아닌 정의된 경로다.

## 합리화 방어

| 변명 | 현실 |
|------|------|
| "너무 단순해서 테스트 불필요" | 단순한 코드도 깨진다. 30초면 된다. |
| "나중에 테스트 쓸게" | 나중에 쓴 테스트는 즉시 통과. 아무것도 증명 못함. |
| "이미 수동 테스트 했다" | 수동은 임시적. 기록 없고 재실행 불가. |
| "삭제하면 낭비" | 매몰 비용 오류. 신뢰할 수 없는 코드가 진짜 낭비. |
| "참고용으로만" | 보면 적응시킨다. 삭제는 삭제다. |
| "탐색이 필요하다" | Spike 경로를 선언하고 밟아라. |
| "TDD가 느리다" | 디버깅이 더 느리다. |

## 체크포인트

config의 `defaults.checkpoint_interval`에 따라 유저에게 진행 보고:
- 완료 태스크 / 전체 태스크, 현재 커버리지, 발견된 이슈, 다음 태스크

## 디버깅 연계

2회 이상 수정 실패 시 → vibe:debugging 스킬 호출.

## Status 업데이트

태스크 완료 시: status의 tasks[N].status=completed, task 번호 증가, coverage 업데이트.
