# 03. 기술 스택

각 선택의 **이유**와 **대안**을 함께 기록. 나중에 바꿀 때 판단 근거가 되도록.

## 언어 / 런타임

| 영역 | 선택 | 이유 |
|---|---|---|
| 백엔드 분석 로직 | **Python 3.11+** | music21, librosa, yt-dlp 등 음악/오디오 라이브러리 생태계가 Python 중심 |
| 사용자 인터페이스 | **Claude Code (에이전트)** | CLI 플래그/HTTP 폼 대신 자연어 요청. 분기/에러 핸들링도 대화로 |
| 악보 뷰어 | **HTML + Vanilla JS (정적 파일)** | 서버 없이 브라우저에서 열기만 하면 됨. 프레임워크 오버킬 |
| 악보 렌더링 | **JavaScript (브라우저)** | OpenSheetMusicDisplay. Node.js 런타임 불필요 |

> 참고: 초기 논의에서 FastAPI 백엔드를 검토했으나, (1) yt-dlp가 클라우드 서버에서 차단당하는 문제, (2) 에이전트가 직접 함수 호출하는 게 자연스럽다는 점, 두 가지 이유로 **HTTP 서버는 제거**. Python은 라이브러리(패키지)로만 존재.

## 패키지 관리

| 영역 | 선택 | 이유 |
|---|---|---|
| Python | **uv** | pip 대비 10~100배 빠름. 가상환경/lock 통합 관리. `uv add`, `uv run` 한 줄로 끝 |
| JS | **CDN 직접 로드** (v0) | OSMD를 `<script src="...">`로 바로 가져오기. 빌드 도구 불필요 |

## 핵심 라이브러리

### Python (staff_ai 패키지)

| 용도 | 라이브러리 | 이유 / 비고 |
|---|---|---|
| YouTube 오디오 추출 | **yt-dlp** | youtube-dl 후속, 활발히 유지보수. 곡 메타데이터(제목, 아티스트)도 함께 추출 |
| 오디오 디코딩 | **ffmpeg** (시스템) + librosa | ffmpeg은 별도 설치 필요 |
| 오디오 분석 | **librosa** | BPM, beat tracking, chroma feature 추출 |
| 코드 진행 추출 | **미정 — 후보 셋 중 선정** | 아래 별도 섹션 |
| 악보 데이터 처리 | **music21** | MIT, MusicXML 생성, 코드 → 음표 변환, 조성 처리 모두 가능 |
| 가사 가져오기 | **httpx** + LRClib API | 별도 라이브러리 없이 직접 호출. `lrclib.net/api/get` |

> 제거됨: FastAPI, uvicorn (HTTP 서버 안 함)

### 프론트엔드 (브라우저)

| 용도 | 라이브러리 | 이유 |
|---|---|---|
| 악보 렌더링 | **OpenSheetMusicDisplay (OSMD)** | MusicXML을 그대로 넣으면 SVG로 그려줌. VexFlow보다 우리 용도에 적합 (XML → 화면 직결) |
| 파일 입력 | **HTML5 File API** | 드래그&드롭으로 MusicXML 파일 받기. 별도 라이브러리 불필요 |

## 결정 보류: 코드 진행 추출 라이브러리

윈도우 호환성 + 한국 노래에서의 정확도가 변수라 **실제 구현 시 1~2개 시도 후 결정**.

| 후보 | 장점 | 단점 |
|---|---|---|
| **autochord** | 순수 Python, 설치 간단 | 정확도 보통 |
| **chord-extractor** (Chordino 래퍼) | 정확도 높음 | Vamp host 시스템 의존, Windows 설치 번거로움 |
| **librosa chroma + 직접 매칭** | 의존성 최소, 완전 제어 | 정확도 직접 튜닝 필요 |
| **omnizart** | ML 기반, 모던 | 모델 무거움, 첫 실행 느림 |

**v0 시도 순서**: `autochord` → 부족하면 `librosa chroma + 직접 매칭` 추가 → 그래도 부족하면 `chord-extractor`.

## 외부 의존성 (시스템)

| 도구 | 용도 | 설치 방법 (Windows) |
|---|---|---|
| **ffmpeg** | yt-dlp/librosa의 오디오 처리 | `winget install Gyan.FFmpeg` |
| **uv** | Python 환경 관리 | `winget install astral-sh.uv` 또는 공식 스크립트 |

## 데이터 포맷

| 단계 | 포맷 | 이유 |
|---|---|---|
| 다운로드된 오디오 | `wav` (모노, 22050Hz) | librosa 처리 표준, 파일 크기 적절 |
| 중간 분석 결과 | Python dict → JSON (캐시용) | 코드 진행, BPM, 박자 등 |
| 가사 | LRC 포맷 → Python list of dict | LRClib 응답을 파싱 |
| 최종 악보 | **MusicXML** (`.musicxml` 파일) | OSMD 표준 입력 포맷, music21이 직접 export |

## 의도적 제외

- **FastAPI / uvicorn / HTTP 서버** — 에이전트가 직접 함수 호출하므로 불필요
- **CLI argparse / click** — 같은 이유
- **Demucs / Spleeter (음원 분리)** — 보컬 멜로디 채보 안 하므로 불필요
- **basic-pitch (멜로디 채보)** — 같은 이유로 불필요
- **데이터베이스** — 캐시는 파일시스템(곡 ID 기반)으로 충분
- **인증/계정** — 개인 로컬용
