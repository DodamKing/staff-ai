# Todo

> 현재 할 일. 완료된 항목은 체크하거나 `log.md`로 옮김.

## 진행 중

(없음)

## 다음 작업 — v0 셋업

> 📌 **셋업은 두 단계로 분리.** 구조/의존성 셋업은 **현재 Windows 11 머신**에서 진행 가능 (uv.lock은 크로스플랫폼). 실제 곡 분석/검증은 **ffmpeg 깔린 환경**에서. 다른 머신은 Windows 10 → `git clone` (OneDrive 밖 경로) → `uv sync` → `winget install Gyan.FFmpeg` 순.

**여기(Win11)에서 가능** ✅ 완료 ([log 2026-05-14](log.md))

- [x] uv 프로젝트 초기화 (`uv init --lib --python 3.11`)
- [x] 기본 의존성 추가: music21, librosa, yt-dlp, httpx (autochord는 vamp 빌드 실패로 보류)
- [x] 폴더 구조 생성: `src/staff_ai/`, `src/staff_ai/patterns/`, `web/`, `outputs/`, `cache/audio/`, `cache/analysis/`
- [x] `src/staff_ai/__init__.py`: `generate_score()` 스텁
- [x] 패키지 import 동작 확인

**다른 환경(ffmpeg 가능한 곳)에서**

- [ ] `git clone` (OneDrive 밖 경로) → `uv sync`
- [ ] ffmpeg 설치 (`winget install Gyan.FFmpeg`)
- [ ] autochord 재시도 (Win10 환경에서 `uv add autochord`) — 안 되면 `librosa chroma + 직접 매칭`으로 전환
- [ ] 실제 곡으로 끝까지 동작 확인 (아래 [검증](#검증) 섹션)

## v0 핵심 작업 (셋업 후)

### Python 모듈

- [ ] `audio.py`: `download_audio(url) -> (wav_path, metadata)` — yt-dlp 호출, 캐시 적용
- [ ] `analysis.py`: `analyze(wav_path) -> dict` — BPM, beats, chords, key
- [ ] `analysis.py`: `choose_pattern(analysis) -> str` — BPM 기반 자동 결정 (block_chord/arpeggio/eight_beat)
- [ ] `lyrics.py`: `fetch_lrc(title, artist) -> list | None` — LRClib API 호출 + LRC 파싱
- [ ] `patterns/block_chord.py`: 코드 → music21 Measure 생성 (v0 기본 패턴)
- [ ] `patterns/__init__.py`: 패턴 레지스트리 (이름 → 함수 매핑)
- [ ] `score.py`: `build_musicxml(...) -> path` — music21 Score 조립, 가사 lyric 태그 매핑, 파일 저장

### 뷰어

- [ ] `web/index.html`: 드래그&드롭 영역 + OSMD 렌더 div
- [ ] `web/app.js`: File API로 .musicxml 읽기 → OSMD에 전달 → 렌더
- [ ] `web/style.css`: 최소 스타일

### 검증

- [ ] 한국 발라드 한 곡으로 끝까지 동작 확인 (Claude Code에서 `generate_score(url)` 호출 → outputs 생성 → 뷰어에 드래그&드롭 → 표시)
- [ ] 코드 진행 정확도 70%+ 확인 (들어보고 판단)
- [ ] 가사가 대략 맞는 마디에 위치하는지 확인

## 보류 / 결정 필요

- [ ] 코드 추정 라이브러리 최종 선정: autochord vs chord-extractor vs librosa+직접 매칭
  - 구현 단계에서 autochord 먼저 시도 → 부족하면 교체
- [ ] LRC 타임스탬프 → 마디 매핑 알고리즘 정밀도
  - v0는 "라인의 시작 시각 ≤ 마디 시작 시각" 그리디 매핑으로 시작

## 백로그 (v1 이후)

[`planning/05-roadmap.md`](planning/05-roadmap.md) 참고.

주요 항목 요약:
- v1: 패턴 추가(arpeggio, eight_beat), `choose_pattern()` 정밀도 개선, 곡 구조 인식
- v2: 키 변경, 가사 음절-음표 정밀 매핑, PDF 다운로드
- v3: 미정 (기타 코드 동시 표시, 모바일 뷰어 등)
