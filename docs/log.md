# 진행 로그

> 날짜별 진행 기록. 위가 최신.
> 작업 완료 단위로 append. todo 갱신만 있는 날은 굳이 기록 안 해도 됨.

---

## 2026-05-14

### 프로젝트 시작 — 설계 문서 작성

**한 일**

- 사용자와 스코프 논의 (여러 라운드)
- 기술 스택 결정: Python + uv + FastAPI + music21 + librosa + OSMD
- 가사 소스로 LRClib 채택 (라인 단위 타임스탬프)
- 가사-악보 통합 방식 (A) 채택: 악보에 lyric 태그로 통합
- `docs/planning/` 5개 문서 작성: 개요/배경/기술스택/아키텍처/로드맵
- 루트 `README.md`, `.gitignore` 추가

**주요 결정**

- 비목표 확정: 음원 재생, 멜로디 채보, 키 변경(v0), 다른 악기, 상용화
- 입력은 YouTube URL 한 가지로 단순화
- 보컬 멜로디 채보 안 함 → 풀밴드 사운드 채보의 가장 어려운 부분 우회
- 코드 진행 + 반주 패턴 알고리즘 생성 방식 채택

### 문서 구조 재정비

**한 일**

- planning 문서들을 `docs/planning/` 하위로 이동 (진행 전 산출물이므로)
- `docs/todo.md`, `docs/log.md` 신설 (진행 중 기록용)
- 루트 `CLAUDE.md` 추가: 작업 규칙 명시
- auto-memory 시스템 사용 중단, 이전에 저장한 메모리 파일 삭제

**주요 결정**

- 모든 프로젝트 맥락/결정/진행상황은 `docs/`에 기록
- auto-memory 시스템은 이 프로젝트에서 사용하지 않음
- 문서 분류: planning(고정) / todo(현재) / log(누적)

### 아키텍처 피벗 — 에이전트 + 정적 뷰어 구조

**배경**

- 배포 시 yt-dlp가 클라우드 서버 IP에서 차단당하는 문제 예상
- "추출은 로컬, 웹은 표시만"으로 분리하면 이 문제 자체가 사라짐
- 사용자가 CLI 플래그보다 자연어 요청(Claude Code)이 편하다고 결정

**한 일** (계획)

- `planning/01-overview.md` 갱신: 에이전트 시나리오 반영
- `planning/03-tech-stack.md` 갱신: FastAPI/uvicorn 제거, Claude Code를 인터페이스로 명시
- `planning/04-architecture.md` 전면 재작성: HTTP 서버 없음, `staff_ai` 패키지 + 정적 뷰어 + 에이전트 호출 구조
- `planning/05-roadmap.md` 갱신: 사용자 패턴 선택 항목 제거 → 자동 결정 정확도 개선으로 재정의
- `docs/todo.md` 갱신: FastAPI 관련 task 제거, 패키지 토대로 변경
- `docs/README.md` 갱신: 코드 진입점이 `__init__.py`로 변경

**주요 결정**

- HTTP 백엔드 서버 만들지 않음 (FastAPI/uvicorn 제외)
- CLI 도구 만들지 않음 (argparse/click 제외)
- 사용자 인터페이스 = Claude Code (자연어 요청)
- 코드 진입점 = `src/staff_ai/__init__.py`의 `generate_score(url)` 함수 하나
- 웹 뷰어 = 백엔드 없는 정적 HTML, 드래그&드롭으로 MusicXML 받음
- 반주 스타일은 **사용자가 선택하지 않음** — 곡 BPM 등으로 자동 결정
  - 이유: 노래 따라 부르려고 만드는 도구라 "발라드를 재즈풍으로" 같은 옵션은 무의미

### git 저장소 + GitHub 연결 + OneDrive 정리

**한 일**

- 로컬 git 저장소 초기화 (`git init`), main 브랜치
- 첫 커밋: 설계 문서 11개 파일 (dodamking 명의)
- GitHub 원격 저장소 연결: https://github.com/DodamKing/staff-ai (private)
- `git push -u origin main` 완료

**OneDrive 정리 (시스템 측면)**

- 현재 머신은 OneDrive 클라이언트가 사실상 비활성 상태였음 (프로세스 미실행, OneDrive.exe PATH 못 찾음)
- Windows 업데이트로 재설치 방지를 위해 그룹 정책 `DisableFileSyncNGSC=1` 적용 (gpedit)
- HKCU의 빈 OneDrive Personal 계정 흔적 제거
- `C:\Users\scsma\OneDrive\Desktop\` 폴더 안 다른 작업물(3.4GB / 12.8만 파일) 존재 → 폴더 자체는 보존, sync만 영구 차단

**다음 환경 셋업**

- 다른 환경(ffmpeg 있는)에서 OneDrive 밖 경로에 `git clone https://github.com/DodamKing/staff-ai.git`
- 이어서 [`docs/todo.md`](../todo.md)의 "다음 작업 — v0 셋업"부터 진행
