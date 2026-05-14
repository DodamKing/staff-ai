# Todo

> 현재 할 일. 완료된 항목은 체크하거나 `log.md`로 옮김.

## 진행 중

(없음)

## 다음 작업 — v0 셋업

> ⚠️ **셋업은 ffmpeg가 이미 깔린 환경에서 시작.** 현재 Windows 머신엔 ffmpeg 없음 → 셋업해도 검증 불가.
>
> ⚠️ **OneDrive 동기화 제외 설정 먼저.** `.venv/`, `cache/`, `outputs/`, `__pycache__/` 폴더는 OneDrive 동기화에서 빼야 충돌 안 남. (`.gitignore`엔 있지만 OneDrive는 별개)

- [ ] OneDrive 동기화 제외: 위 4개 폴더를 OneDrive 설정에서 제외 (또는 폴더 우클릭 → "이 디바이스에 항상 보관" 끄기)
- [ ] uv 프로젝트 초기화 (`uv init`, Python 3.11+)
- [ ] 기본 의존성 추가: music21, librosa, yt-dlp, httpx, autochord
- [ ] ffmpeg 설치 확인 (Windows: `winget install Gyan.FFmpeg`)
- [ ] 폴더 구조 생성: `src/staff_ai/`, `src/staff_ai/patterns/`, `web/`, `outputs/`, `cache/audio/`, `cache/analysis/`
- [ ] `src/staff_ai/__init__.py`: `generate_score()` 스텁 작성 (의존 모듈들 import만)
- [ ] 패키지 import 동작 확인 (`uv run python -c "from staff_ai import generate_score"`)

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
