# 04. 아키텍처

## 구조 개요

세 부분으로 분리:

1. **Python 패키지** (`staff_ai`) — 로컬 실행, 분석/악보 생성 toolkit. HTTP 서버 아님.
2. **에이전트** (Claude Code) — 사용자 자연어 요청을 받아 toolkit 함수 호출.
3. **정적 웹 뷰어** (`web/`) — 생성된 MusicXML을 브라우저에서 렌더. 백엔드 없음.

## 데이터 흐름

```
[사용자]
   │ "이 URL 악보 만들어줘 [URL]" (자연어, Claude Code)
   ▼
[Claude Code (에이전트)]
   │ from staff_ai import generate_score
   │ generate_score(url) 호출
   ▼
[staff_ai 패키지 (로컬 Python)]
   │
   ├─▶ audio.download_audio(url)
   │       → cache/audio/{video_id}.wav
   │       → metadata: {title, artist, duration}
   │
   ├─▶ analysis.analyze(wav_path)
   │       → {chords: [...], bpm, beats, key, time_signature}
   │       (autochord + librosa)
   │
   ├─▶ analysis.choose_pattern(bpm, key, ...)
   │       → "block_chord" | "arpeggio" | "8beat" 등 (자동 결정)
   │
   ├─▶ lyrics.fetch_lrc(title, artist)
   │       → [{time: 23.45, text: "사랑한다고..."}, ...]
   │       (실패 시 None — 가사 없이 진행)
   │
   └─▶ score.build_musicxml(analysis, lyrics, pattern)
              → music21 Score 객체
              → outputs/{title}.musicxml 저장
              → 파일 경로 반환
                       │
                       ▼
[사용자]
   │ 브라우저로 web/index.html 열기
   │ outputs/{title}.musicxml 드래그&드롭
   ▼
[OpenSheetMusicDisplay] → SVG 렌더
```

## 폴더 구조 (목표)

```
staff-ai/
├── docs/                       # 문서
│   ├── README.md               # 문서 진입점
│   ├── todo.md
│   ├── log.md
│   └── planning/               # 설계 문서 (이 파일 포함)
│       └── 01~05-*.md
│
├── src/staff_ai/               # Python 패키지
│   ├── __init__.py             # ★ 코드 진입점 — generate_score() 노출
│   ├── audio.py                # YouTube → wav + 메타데이터
│   ├── analysis.py             # 코드/BPM 추출 + 패턴 자동 결정
│   ├── lyrics.py               # LRClib 호출
│   ├── score.py                # music21 MusicXML 생성
│   └── patterns/               # 반주 패턴 정의
│       ├── __init__.py         # 패턴 레지스트리
│       ├── block_chord.py
│       ├── arpeggio.py
│       └── eight_beat.py
│
├── web/                        # 정적 뷰어 (서버 없음, 그냥 브라우저로 열기)
│   ├── index.html              # 드래그&드롭 영역 + OSMD 렌더 영역
│   ├── app.js                  # OSMD 초기화, 파일 로드 처리
│   └── style.css
│
├── outputs/                    # 생성된 MusicXML (gitignore)
│   └── *.musicxml
│
├── cache/                      # 다운로드/분석 결과 캐시 (gitignore)
│   ├── audio/                  # *.wav
│   └── analysis/               # *.json
│
├── pyproject.toml              # uv가 관리
├── uv.lock
├── .gitignore
├── CLAUDE.md                   # 작업 규칙
└── README.md                   # → docs/README.md 안내
```

## 진입점

### 코드 진입점: `src/staff_ai/__init__.py`

여기서 단일 함수 노출. 에이전트(또는 사용자)는 이것만 호출하면 끝.

```python
# src/staff_ai/__init__.py (예정 골격)
from .audio import download_audio
from .analysis import analyze, choose_pattern
from .lyrics import fetch_lrc
from .score import build_musicxml

def generate_score(url: str, output_dir: str = "outputs") -> str:
    """
    YouTube URL → MusicXML 파일 경로.

    Returns: 저장된 .musicxml 파일의 절대 경로.
    """
    wav_path, meta = download_audio(url)
    analysis = analyze(wav_path)
    pattern = choose_pattern(analysis)
    lyrics = fetch_lrc(meta["title"], meta["artist"])
    out_path = build_musicxml(analysis, lyrics, pattern, meta, output_dir)
    return out_path
```

에이전트 사용 예시:

```python
# Claude Code가 한 줄로 호출
from staff_ai import generate_score
path = generate_score("https://youtu.be/...")
print(f"악보 저장됨: {path}")
```

### 사용자 진입점: Claude Code 대화 + 브라우저 뷰어

1. **요청**: Claude Code에서 "이 URL 악보 만들어줘 [URL]"
2. **확인**: 생성 완료 후 Claude가 파일 경로 알려줌
3. **표시**: 브라우저로 `web/index.html` 열고 파일 드래그&드롭

### 문서 진입점: `docs/README.md`

## 모듈 책임 (관심사 분리)

각 모듈은 **하나의 책임만** 가짐. 테스트/교체 용이.

| 모듈 | 책임 | 입력 | 출력 |
|---|---|---|---|
| `audio.py` | YouTube URL → 로컬 wav + 메타데이터 | URL | (wav 경로, {title, artist, duration}) |
| `analysis.py` | 오디오 → 음악 정보 + 패턴 결정 | wav 경로 | {chords, bpm, beats, key, time_sig}, pattern_name |
| `lyrics.py` | 곡명/아티스트 → LRC 가사 | (제목, 아티스트) | `[{time, text}, ...]` 또는 None |
| `score.py` | 음악 정보 + 가사 → MusicXML 파일 | 위 결과들 | 저장된 파일 경로 |
| `patterns/*.py` | 코드 → 마디 음표 시퀀스 | 코드 심볼, 박자 | `music21.stream.Measure` |
| `__init__.py` | 전체 파이프라인 조합 | URL | MusicXML 파일 경로 |

## 패턴 자동 결정 로직 (analysis.choose_pattern)

사용자 선택 없이 곡 특성에서 자동 결정. v0는 단순 임계값:

```
if bpm < 90:        → arpeggio  (잔잔, 아르페지오)
elif bpm < 130:     → block_chord  (중간, 블록코드)
else:               → eight_beat  (빠름, 8비트)
```

v1 이후: 코드 변화 빈도, 키의 메이저/마이너, 곡 구조(verse/chorus) 등 반영해서 정밀도 개선.

## 캐시 전략

같은 YouTube URL 재요청 시 다시 다운로드/분석하지 않음.

- 캐시 키: YouTube video ID (URL에서 추출)
- 저장 위치: `cache/audio/{video_id}.wav`, `cache/analysis/{video_id}.json`
- 무효화: 수동 (디렉토리 삭제)

## 웹 뷰어 동작

`web/index.html`은 **백엔드 없는 순수 정적 페이지.**

1. 사용자가 브라우저로 `web/index.html` 직접 열기 (file:// 또는 GitHub Pages 등)
2. 화면에 드래그&드롭 영역 표시
3. `outputs/곡명.musicxml` 파일을 끌어다 놓기
4. JavaScript가 파일을 읽어 OSMD에 전달 → SVG로 렌더

→ `fetch()`로 outputs 폴더에서 자동 로드하지 않는 이유: file:// 프로토콜에서 보안 제약. 드래그&드롭은 file:// 에서도 동작.

→ 미래에 작은 편의를 위해 `python -m http.server` 한 줄로 띄우는 옵션은 README에 안내만 추가 (v1 이상에서).

## 의도적 단순화

- **HTTP 서버 없음** — 에이전트가 직접 함수 호출 + 정적 뷰어
- **CLI 없음** — 에이전트 인터페이스로 충분
- **인증 없음** — 개인 로컬용
- **데이터베이스 없음** — 파일시스템 캐시로 충분
- **백그라운드 작업 큐 없음** — 동기 처리 (수십 초 걸리면 에이전트 응답 기다림)
- **에러 핸들링 최소** — 예외 그대로 던짐. 에이전트가 보고 대응 (예: "가사 없으면 그냥 진행할까요?")
