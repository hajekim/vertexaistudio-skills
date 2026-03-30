# vertexai-skills

Vertex AI 개발을 위한 **Gemini CLI Skills** 모음. `google-genai` Python SDK 기준으로 LLM 텍스트 생성, 이미지 생성/편집, 비디오 생성의 올바른 코드 패턴과 레퍼런스를 제공한다.

---

## 스킬 목록

| 스킬 | 파일 | 다루는 내용 |
|------|------|------------|
| `vertexai-skills` | `skills/vertexai-skills/SKILL.md` | 텍스트 생성, 시스템 지시, 함수 호출, 구조화된 출력, 생성 파라미터, 코드 실행 |
| `image-skills` | `skills/image-skills/SKILL.md` | Imagen 4 이미지 생성, Gemini 이미지 생성/편집, Best Practices, Limitations, Safety |
| `video-skills` | `skills/video-skills/SKILL.md` | Veo 텍스트→비디오, 이미지→비디오, 프레임 보간, 비디오 연장, 고급 기법, Safety |

---

## 사전 요구사항

### 라이브러리

```bash
pip install --upgrade google-genai        # 모든 스킬 공통
pip install --upgrade pillow              # image-skills: 이미지 파일 저장 시 필요
```

> 검증된 버전: `google-genai 1.69.0` (2026-03 기준)

### 필수 환경변수

```bash
export GOOGLE_CLOUD_PROJECT="your-project-id"
export GOOGLE_CLOUD_LOCATION="us-central1"   # "global"은 일부 모델만 지원
export GOOGLE_GENAI_USE_VERTEXAI="True"
```

### GCS 버킷 (video-skills 전용)

비디오 생성 결과는 반드시 Cloud Storage에 저장된다. 미리 버킷을 생성해야 한다:

```bash
gcloud storage buckets create gs://your-bucket --location=us-central1
export OUTPUT_GCS_URI="gs://your-bucket/output/"
```

### 인증

```bash
gcloud auth application-default login
```

---

## 설치 (심볼릭 링크 배포)

스킬 파일은 프로젝트 내 `skills/` 디렉토리에서 관리하고, Gemini CLI가 읽을 수 있도록 `~/.gemini/skills/`에 심볼릭 링크를 생성한다.

```bash
# 프로젝트 클론
git clone <repo-url> ~/sandbox/vertexai-skills
cd ~/sandbox/vertexai-skills

# Gemini CLI skills 디렉토리 생성 (없을 경우)
mkdir -p ~/.gemini/skills

# 심볼릭 링크 생성
ln -s "$(pwd)/skills/vertexai-skills" ~/.gemini/skills/vertexai-skills
ln -s "$(pwd)/skills/image-skills"    ~/.gemini/skills/image-skills
ln -s "$(pwd)/skills/video-skills"    ~/.gemini/skills/video-skills
```

### 설치 확인

```bash
ls -la ~/.gemini/skills/
```

정상 출력 예시:
```
image-skills    -> /home/user/sandbox/vertexai-skills/skills/image-skills
vertexai-skills -> /home/user/sandbox/vertexai-skills/skills/vertexai-skills
video-skills    -> /home/user/sandbox/vertexai-skills/skills/video-skills
```

---

## 사용 방법

스킬은 Gemini CLI가 **자동으로 로드**한다. 별도 명령어 없이 아래 키워드로 요청하면 해당 스킬이 활성화된다.

### vertexai-skills 트리거

| 트리거 | 예시 요청 |
|--------|---------|
| 코드에 `from google import genai` 포함 | "이 코드에 스트리밍을 추가해줘" |
| `Vertex AI`, `Gemini on Vertex` | "Vertex AI로 텍스트 생성 코드 작성해줘" |
| `function calling`, `함수 호출` | "날씨 API를 함수 호출로 연결하는 코드 써줘" |
| `structured output`, `구조화된 출력` | "JSON 형식으로 응답받는 코드 써줘" |
| `code execution`, `코드 실행` | "모델이 직접 계산하는 코드 실행 예제 써줘" |
| `system instruction`, `시스템 지시` | "시스템 지시로 역할을 설정하는 방법 알려줘" |

### image-skills 트리거

| 트리거 | 예시 요청 |
|--------|---------|
| `Imagen`, `아이마젠` | "Imagen 4로 이미지 생성하는 코드 써줘" |
| `image generation`, `이미지 생성` | "이미지 생성하는 코드 만들어줘" |
| `edit image`, `이미지 편집` | "기존 이미지를 수채화 스타일로 편집해줘" |
| `Gemini image`, `제미나이 이미지` | "Gemini 이미지 모델로 생성하는 방법 알려줘" |

### video-skills 트리거

| 트리거 | 예시 요청 |
|--------|---------|
| `Veo`, `비오` | "Veo로 비디오 생성하는 코드 써줘" |
| `video generation`, `비디오 생성` | "텍스트로 영상 만드는 코드 써줘" |
| `text to video`, `텍스트로 영상` | "프롬프트로 비디오 생성해줘" |
| `image to video`, `이미지로 영상` | "이미지를 비디오로 변환하는 코드 써줘" |
| `extend video`, `영상 연장` | "비디오 연장하는 방법 알려줘" |

---

## 스킬 내용 구조

### vertexai-skills (352줄)

```
환경 설정
  - 라이브러리 설치
  - 환경변수
  - 클라이언트 초기화 (HttpOptions api_version="v1")
§ 1. 텍스트 생성         — 논스트리밍 / 스트리밍 채팅
§ 2. 시스템 지시          — GenerateContentConfig.system_instruction
§ 3. 함수 호출            — FunctionDeclaration → Tool → function_response 루프
§ 4. 구조화된 출력        — response_mime_type + response_schema (JSON Schema)
§ 5. 생성 파라미터        — temperature, top_p, top_k, max_output_tokens 등
§ 6. 코드 실행            — ToolCodeExecution, executable_code, code_execution_result
```

### image-skills (323줄)

```
환경 설정
  - 라이브러리 설치 (google-genai + pillow)
  - 모델 선택 가이드 (Imagen 4 vs Gemini Image)
§ 1. Imagen 4 이미지 생성  — generate_images() + GenerateImagesConfig
§ 2. Gemini 이미지 생성    — generate_content() + Modality.IMAGE
§ 3. Gemini 이미지 편집    — 이미지 입력 + 멀티턴 편집
§ 4. Best Practices        — 구체적 프롬프트, 카메라 용어, 반복 개선
§ 5. Limitations           — 언어 지원, 입력 제한, 출력 한계
§ 6. Responsible AI & Safety — 금지 콘텐츠, finish_reason SAFETY 처리
```

### video-skills (523줄)

```
환경 설정
  - GCS 버킷 필수 안내
  - 모델 목록 (veo-3.1-generate-001 권장)
  - 공통 폴링 루프 패턴 (LRO)
§ 1. 텍스트 → 비디오       — generate_videos() + GenerateVideosConfig
§ 2. 이미지 → 비디오       — Image(gcs_uri=...) 입력 패턴
§ 3. 첫/마지막 프레임      — image + last_frame 파라미터
§ 4. 비디오 연장            — Video(uri=..., mime_type="video/mp4")
§ 5. 고급 기법              — 레퍼런스 이미지, 객체 삽입/제거 (veo-2.0 전용)
§ 6. 프롬프트 가이드       — 7가지 구성 요소, 발화 표현, enhance_prompt
§ 7. Responsible AI & Safety — 금지 콘텐츠, operation.error 처리
```

---

## 주요 모델 레퍼런스

### LLM

| 모델 | 특징 |
|------|------|
| `gemini-2.5-flash` | 빠른 텍스트 생성 (권장) |
| `gemini-2.5-pro` | 고품질 복잡한 추론 |

### 이미지

| 모델 | API | 특징 |
|------|-----|------|
| `imagen-4.0-generate-001` | `generate_images()` | 최고 품질 정적 이미지 |
| `gemini-2.5-flash-image` | `generate_content()` | 생성 + 편집, 경량 |
| `gemini-3-pro-image-preview` | `generate_content()` | 생성 + 편집, 최고 품질 |
| `gemini-3.1-flash-image-preview` | `generate_content()` | 생성 + 편집, 최신 |

### 비디오

| 모델 | 특징 |
|------|------|
| `veo-3.1-generate-001` | 최신 안정 버전 **(권장)** |
| `veo-3.1-fast-generate-001` | 빠른 생성 |
| `veo-3.0-generate-001` | 이전 안정 버전 |
| `veo-2.0-generate-001` | 레거시 (객체 삽입/제거, enhance_prompt 전용) |

---

## 공통 클라이언트 초기화 패턴

세 스킬 모두 동일한 초기화 패턴을 사용한다:

```python
from google import genai
from google.genai.types import HttpOptions

client = genai.Client(http_options=HttpOptions(api_version="v1"))
```

> `client`는 모듈 최상단에서 한 번만 초기화한다. 함수마다 재생성하지 않는다.

대안 (환경변수 대신 직접 지정):

```python
client = genai.Client(
    vertexai=True,
    project="your-project-id",
    location="us-central1",
)
```

---

## 프로젝트 구조

```
vertexai-skills/
├── skills/
│   ├── vertexai-skills/
│   │   └── SKILL.md          # LLM 텍스트 생성 레퍼런스
│   ├── image-skills/
│   │   └── SKILL.md          # 이미지 생성/편집 레퍼런스
│   └── video-skills/
│       └── SKILL.md          # 비디오 생성 레퍼런스
└── docs/
    └── superpowers/
        ├── specs/            # 설계 명세서
        └── plans/            # 구현 계획서

# 배포 (심볼릭 링크)
~/.gemini/skills/vertexai-skills  →  skills/vertexai-skills/
~/.gemini/skills/image-skills     →  skills/image-skills/
~/.gemini/skills/video-skills     →  skills/video-skills/
```

---

## 스킬 업데이트

스킬 파일은 git으로 관리된다. 심볼릭 링크는 항상 최신 파일을 가리키므로 **별도 재배포 없이** 수정 즉시 반영된다.

```bash
# 스킬 수정 후
git add skills/<skill-name>/SKILL.md
git commit -m "feat: update <section> in <skill-name>"
# 완료 — Gemini CLI는 다음 실행 시 자동으로 업데이트된 내용을 로드
```

---

## SDK 참고 문서

- [google-genai Python SDK](https://googleapis.github.io/python-genai/)
- [Vertex AI Gemini API](https://cloud.google.com/vertex-ai/generative-ai/docs/learn/overview)
- [Imagen on Vertex AI](https://cloud.google.com/vertex-ai/generative-ai/docs/image/overview)
- [Veo on Vertex AI](https://cloud.google.com/vertex-ai/generative-ai/docs/video/overview)
