# staff-ai 작업 규칙

이 프로젝트는 **모든 맥락/결정/진행상황을 `docs/`에 기록**합니다.

## 메모리 시스템 사용 금지

- auto-memory 시스템(`~/.claude/projects/.../memory/`)에 이 프로젝트 내용 저장하지 말 것
- 프로젝트 관련된 모든 것은 `docs/`에 문서화

## 문서 구조

| 위치 | 역할 | 변경 빈도 |
|---|---|---|
| `docs/README.md` | 문서 진입점, 목차 | 드물게 |
| `docs/planning/` | 진행 시작 전 만든 설계 문서 (개요/배경/기술스택/아키텍처/로드맵) | 거의 안 바뀜 |
| `docs/todo.md` | 현재 할 일 | 자주 |
| `docs/log.md` | 진행 기록 (날짜별 append) | 작업 단위마다 |

## 작업 흐름

1. 새 작업 시작 전 `docs/todo.md`, `docs/log.md` 확인
2. 작업 중 todo 업데이트 (완료 체크, 새 항목 추가)
3. 작업 완료 시 `docs/log.md`에 해당 날짜 섹션 append
4. 큰 결정/스코프 변경은 `docs/planning/` 문서 업데이트 또는 별도 ADR 추가 검토
