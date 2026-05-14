# staff-ai 문서

> 진입점. 여기서 시작.

## 구조

| 위치 | 내용 | 변경 빈도 |
|---|---|---|
| [`planning/`](planning/) | 진행 시작 전 만든 설계 문서 (개요/배경/기술스택/아키텍처/로드맵) | 거의 안 바뀜 |
| [`todo.md`](todo.md) | 현재 할 일 | 자주 |
| [`log.md`](log.md) | 진행 기록 (날짜별 append) | 작업 단위마다 |

## 빠른 요약

- **입력**: YouTube URL (대중가요)
- **출력**: 피아노 반주 악보 (가사 포함) — 브라우저에 표시
- **현재 단계**: v0 시작 전, 설계 문서 완료
- **다음 작업**: [`todo.md`](todo.md)

## planning 문서 목차

설계 시점에 정한 큰 그림. 큰 방향이 바뀌면 여기 업데이트.

| 문서 | 내용 |
|---|---|
| [planning/01-overview.md](planning/01-overview.md) | 개요 — 뭘 만드는지 |
| [planning/02-background.md](planning/02-background.md) | 배경 — 왜 만드는지 |
| [planning/03-tech-stack.md](planning/03-tech-stack.md) | 기술 스택 + 선택 이유 |
| [planning/04-architecture.md](planning/04-architecture.md) | 폴더 구조, 데이터 흐름, 진입점 |
| [planning/05-roadmap.md](planning/05-roadmap.md) | v0/v1/v2/v3 단계 계획 |

## 진입점

- **문서 진입점**: 이 파일 (`docs/README.md`)
- **코드 진입점** (예정): `src/staff_ai/__init__.py` — `generate_score(url)` 함수
- **사용자 진입점**: Claude Code에 자연어 요청 ("이 URL 악보 만들어줘 [URL]")
- **뷰어 진입점**: 브라우저로 `web/index.html` 열어서 생성된 `.musicxml` 드래그&드롭

## 작업 규칙

루트 [`CLAUDE.md`](../CLAUDE.md) 참고. 요약:
- auto-memory 사용 안 함, 모든 맥락은 docs에 기록
- 작업 시작 전 todo/log 확인 → 진행 중 todo 갱신 → 완료 시 log에 append
