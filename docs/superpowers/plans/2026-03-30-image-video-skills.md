# Image & Video Skills Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** `image-skills/SKILL.md`과 `video-skills/SKILL.md` 두 Gemini CLI 스킬 파일을 작성하고 `~/.gemini/skills/`에 심볼릭 링크로 배포.

**Architecture:** 각 스킬은 독립 파일로 작성 후 `~/.gemini/skills/`에 심볼릭 링크. image-skills는 Imagen 4와 Gemini Image 두 API 패턴을 분리 서술. video-skills는 Veo 3.1 기준, 폴링 루프를 모든 생성 섹션의 공통 패턴으로 사용.

**Tech Stack:** `google-genai` Python SDK, Pillow (이미지 처리), Gemini CLI skills 시스템 (SKILL.md frontmatter)

---

## File Map

| 파일 | 역할 |
|------|------|
| `skills/image-skills/SKILL.md` | 이미지 생성/편집 스킬 소스 |
| `skills/video-skills/SKILL.md` | 비디오 생성 스킬 소스 |
| `~/.gemini/skills/image-skills` → `skills/image-skills/` | Gemini CLI 배포 링크 |
| `~/.gemini/skills/video-skills` → `skills/video-skills/` | Gemini CLI 배포 링크 |

---

# PART A: image-skills

---

## Task 1: image-skills — frontmatter + 환경 설정

**Files:**
- Create: `skills/image-skills/SKILL.md`

- [ ] **Step 1: 디렉토리 생성 및 SKILL.md 작성**

```bash
mkdir -p /home/ext_hajekim_google_com/sandbox/vertexai-skills/skills/image-skills
```

`skills/image-skills/SKILL.md`를 아래 내용으로 생성:

````markdown
---
name: image-skills
description: Use when working with image generation or editing on Vertex AI using google-genai Python SDK — imports Imagen or Gemini image models, or requests involving "image generation", "generate image", "edit image", "Imagen", "Gemini image". 한국어: "이미지 생성", "이미지 편집", "이미지 만들어줘", "아이마젠", "제미나이 이미지".
version: 1.0.0
---

# Image Skills — google-genai Python SDK 레퍼런스

## 환경 설정

### 라이브러리 설치

```bash
pip install --upgrade google-genai pillow   # pillow: 이미지 파일 저장에 필요
```

### 필수 환경변수

```bash
export GOOGLE_CLOUD_PROJECT="your-project-id"
export GOOGLE_CLOUD_LOCATION="us-central1"
export GOOGLE_GENAI_USE_VERTEXAI="True"
```

### 클라이언트 초기화

```python
from google import genai

client = genai.Client()
```

### 모델 선택 가이드

| 모델 | 용도 | API |
|------|------|-----|
| `imagen-4.0-generate-001` | 최고 품질 정적 이미지 | `generate_images()` |
| `gemini-2.5-flash-image` | 생성 + 편집, 경량 | `generate_content()` |
| `gemini-3-pro-image-preview` | 생성 + 편집, 최고 품질 | `generate_content()` |
| `gemini-3.1-flash-image-preview` | 생성 + 편집, 최신 경량 | `generate_content()` |

> **선택 기준:**
> - 고품질 정적 이미지만 필요 → Imagen 4
> - 편집, 멀티턴, 텍스트+이미지 혼합 → Gemini Image 모델
````

- [ ] **Step 2: 파일 존재 확인**

```bash
head -20 /home/ext_hajekim_google_com/sandbox/vertexai-skills/skills/image-skills/SKILL.md
```

Expected: frontmatter와 환경 설정 섹션 출력.

- [ ] **Step 3: Commit**

```bash
cd /home/ext_hajekim_google_com/sandbox/vertexai-skills
git add skills/image-skills/SKILL.md
git commit -m "feat: scaffold image-skills SKILL.md with env setup"
```

---

## Task 2: image-skills — § 1 Imagen 4 이미지 생성

**Files:**
- Modify: `skills/image-skills/SKILL.md`

- [ ] **Step 1: § 1 섹션 파일 끝에 추가**

아래 내용을 `skills/image-skills/SKILL.md` 끝에 추가 (파일 전체를 다시 쓸 것):

````markdown

---

## § 1. Imagen 4 이미지 생성

### 언제: 최고 품질의 정적 이미지가 필요할 때

```python
from google import genai
from google.genai.types import GenerateImagesConfig

client = genai.Client()

response = client.models.generate_images(
    model="imagen-4.0-generate-001",
    prompt="A serene mountain lake at sunset with reflections",
    config=GenerateImagesConfig(
        number_of_images=1,        # 생성할 이미지 수 (1-4)
        aspect_ratio="1:1",        # "1:1", "4:3", "3:4", "16:9", "9:16"
        language="ko",             # 프롬프트 언어 힌트 (선택)
    ),
)

# 이미지 저장
for i, generated in enumerate(response.generated_images):
    generated.image.save(f"output_{i}.png")
    print(f"Saved output_{i}.png")
```

> **원칙:**
> - `generate_images()`는 Imagen 전용 API다. Gemini 모델에는 사용할 수 없다.
> - 결과는 `response.generated_images` 리스트로 반환된다.
> - 저장 시 `pillow` 라이브러리가 필요하다.
````

- [ ] **Step 2: 섹션 헤더 확인**

```bash
grep "^## " /home/ext_hajekim_google_com/sandbox/vertexai-skills/skills/image-skills/SKILL.md
```

Expected:
```
## 환경 설정
## § 1. Imagen 4 이미지 생성
```

- [ ] **Step 3: Commit**

```bash
cd /home/ext_hajekim_google_com/sandbox/vertexai-skills
git add skills/image-skills/SKILL.md
git commit -m "feat: add §1 Imagen 4 image generation to image-skills"
```

---

## Task 3: image-skills — § 2 Gemini 이미지 생성

**Files:**
- Modify: `skills/image-skills/SKILL.md`

- [ ] **Step 1: § 2 섹션 추가**

````markdown

---

## § 2. Gemini 이미지 생성

### 언제: 텍스트 설명과 함께 이미지가 필요하거나 멀티모달 응답이 필요할 때

```python
from google import genai
from google.genai.types import GenerateContentConfig, Modality
from PIL import Image
from io import BytesIO
import os

client = genai.Client()

response = client.models.generate_content(
    model="gemini-2.5-flash-image",
    contents="Generate an image of the Eiffel tower with fireworks in the background.",
    config=GenerateContentConfig(
        response_modalities=[Modality.TEXT, Modality.IMAGE],
    ),
)

os.makedirs("output", exist_ok=True)
for i, part in enumerate(response.candidates[0].content.parts):
    if part.text:
        print(part.text)
    elif part.inline_data:
        image = Image.open(BytesIO(part.inline_data.data))
        image.save(f"output/image_{i}.png")
        print(f"Saved output/image_{i}.png")
```

### 고품질 모델 사용 시

```python
response = client.models.generate_content(
    model="gemini-3-pro-image-preview",   # 최고 품질
    contents="Generate a photorealistic portrait of a golden retriever in a park.",
    config=GenerateContentConfig(
        response_modalities=[Modality.TEXT, Modality.IMAGE],
    ),
)
```

> **원칙:**
> - `response_modalities=[Modality.TEXT, Modality.IMAGE]`는 필수다.
> - 응답 파트를 순회하며 `part.text`와 `part.inline_data`를 분기 처리한다.
> - 이미지 데이터는 `part.inline_data.data` (bytes)로 반환된다.
````

- [ ] **Step 2: Commit**

```bash
cd /home/ext_hajekim_google_com/sandbox/vertexai-skills
git add skills/image-skills/SKILL.md
git commit -m "feat: add §2 Gemini image generation to image-skills"
```

---

## Task 4: image-skills — § 3 Gemini 이미지 편집

**Files:**
- Modify: `skills/image-skills/SKILL.md`

- [ ] **Step 1: § 3 섹션 추가**

````markdown

---

## § 3. Gemini 이미지 편집

### 언제: 기존 이미지를 수정하거나 스타일을 변환해야 할 때

```python
from google import genai
from google.genai.types import GenerateContentConfig, Modality
from PIL import Image
from io import BytesIO

client = genai.Client()

# 원본 이미지 로드
source_image = Image.open("input.png")

response = client.models.generate_content(
    model="gemini-3-pro-image-preview",
    contents=[source_image, "Edit this image to make it look like a watercolor painting."],
    config=GenerateContentConfig(
        response_modalities=[Modality.TEXT, Modality.IMAGE],
    ),
)

for part in response.candidates[0].content.parts:
    if part.text:
        print(part.text)
    elif part.inline_data:
        edited = Image.open(BytesIO(part.inline_data.data))
        edited.save("edited.png")
        print("Saved edited.png")
```

### 멀티턴 편집 (순차적 수정)

```python
chat = client.chats.create(
    model="gemini-3-pro-image-preview",
    config=GenerateContentConfig(
        response_modalities=[Modality.TEXT, Modality.IMAGE],
    ),
)

# 1차 편집
source_image = Image.open("input.png")
response1 = chat.send_message([source_image, "Make the sky more dramatic with dark clouds."])

# 2차 편집 (이전 결과에 이어서)
response2 = chat.send_message("Now add lightning bolts in the sky.")

for part in response2.candidates[0].content.parts:
    if part.inline_data:
        Image.open(BytesIO(part.inline_data.data)).save("final.png")
```

> **원칙:**
> - 편집 프롬프트에 원본 이미지를 `contents` 리스트 첫 번째 요소로 전달한다.
> - 멀티턴 편집은 `client.chats.create()`로 세션을 유지한다.
> - 요청 전체 파일 크기는 50MB 이하로 유지한다.
````

- [ ] **Step 2: Commit**

```bash
cd /home/ext_hajekim_google_com/sandbox/vertexai-skills
git add skills/image-skills/SKILL.md
git commit -m "feat: add §3 Gemini image editing to image-skills"
```

---

## Task 5: image-skills — § 4 Best Practices

**Files:**
- Modify: `skills/image-skills/SKILL.md`

- [ ] **Step 1: § 4 섹션 추가**

````markdown

---

## § 4. Best Practices

### 프롬프트 작성 원칙

**구체적으로 묘사한다**
```
# 나쁜 예
"a warrior"

# 좋은 예
"A medieval elf archer with leather arm guards, fingerless gloves, and a wooden quiver"
```

**목적을 명시한다**
```
# 나쁜 예
"make a logo"

# 좋은 예
"A minimalist premium logo for a skincare brand, clean lines, pastel tones"
```

**긍정적 표현을 사용한다**
```
# 나쁜 예
"no cars in the street"

# 좋은 예
"an empty cobblestone street"
```

### 카메라/구도 용어 활용

| 목적 | 용어 예시 |
|------|---------|
| 넓은 장면 | `wide-angle shot`, `establishing shot` |
| 인물 클로즈업 | `close-up portrait`, `macro shot` |
| 극적 분위기 | `low-angle shot`, `Dutch angle` |
| 자연스러운 느낌 | `eye-level shot`, `candid style` |

### 반복적 개선

초기 결과가 완벽하지 않으면 후속 프롬프트로 조정한다:
```
"Make the lighting warmer"
"Make the expression more serious"
"Add more detail to the background"
```

### 단계별 구성 (복잡한 장면)

복잡한 이미지는 순서대로 지시한다:
1. 배경 설정: `"A foggy Victorian street at night"`
2. 주요 피사체 추가: `"Add a detective in a trench coat"`
3. 세부 요소 추가: `"Add gas lamps and wet cobblestones"`

> **원칙:** 프롬프트가 모호하면 이미지 없이 텍스트만 반환될 수 있다. 이미지 생성을 명시적으로 요청한다.
````

- [ ] **Step 2: Commit**

```bash
cd /home/ext_hajekim_google_com/sandbox/vertexai-skills
git add skills/image-skills/SKILL.md
git commit -m "feat: add §4 best practices to image-skills"
```

---

## Task 6: image-skills — § 5 Limitations + § 6 Responsible AI

**Files:**
- Modify: `skills/image-skills/SKILL.md`

- [ ] **Step 1: § 5, § 6 섹션 추가**

````markdown

---

## § 5. Limitations

### 언어 지원

| 모델 | 지원 언어 |
|------|---------|
| `gemini-2.5-flash-image` | EN, es-MX, ja-JP, zh-CN, hi-IN |
| `gemini-3-pro-image-preview` | AR, DE, EN, ES, FR, HI, ID, IT, JA, KO, PT, RU, UK, VI, ZH |

### 입력 제한

- 오디오/비디오 입력 불가 (이미지 + 텍스트만 지원)
- 요청당 최대 이미지 수: Gemini 2.5 Flash → 3개 / Gemini 3 Pro → 14개
- 전체 요청 크기: 50MB 이하

### 출력 한계

- 요청한 이미지 수보다 적게 생성될 수 있음 (안전 필터로 인해)
- 이미지 내 텍스트 생성 시: 텍스트 먼저 생성 → 이미지 생성 2단계 접근 필요
- 프롬프트가 모호하면 이미지 없이 텍스트만 반환될 수 있음 → 명시적으로 이미지 요청

---

## § 6. Responsible AI & Safety

### 금지 콘텐츠

다음 카테고리는 Generative AI Prohibited Use Policy에 의해 금지됨:
- 아동 착취 콘텐츠
- 폭력적 극단주의 / 테러리즘
- 비동의 친밀 이미지
- 자해 조장
- 성적으로 노골적인 콘텐츠
- 혐오 발언
- 괴롭힘 / 사이버불링

### Safety 필터 에러 처리

```python
response = client.models.generate_content(
    model="gemini-3-pro-image-preview",
    contents="Generate an image...",
    config=GenerateContentConfig(
        response_modalities=[Modality.TEXT, Modality.IMAGE],
    ),
)

# 안전 필터 차단 여부 확인
candidate = response.candidates[0]
if candidate.finish_reason.name == "SAFETY":
    print("Safety filter blocked this request")
    for rating in candidate.safety_ratings:
        if rating.blocked:
            print(f"Blocked by: {rating.category}")
else:
    # 정상 처리
    for part in candidate.content.parts:
        if part.inline_data:
            Image.open(BytesIO(part.inline_data.data)).save("output.png")
```

> **원칙:**
> - `finish_reason`이 `SAFETY`이면 프롬프트를 수정해야 한다.
> - 안전 필터 민감도는 애플리케이션 요구사항에 따라 조정 가능하다.
````

- [ ] **Step 2: 전체 섹션 구조 확인**

```bash
grep "^## " /home/ext_hajekim_google_com/sandbox/vertexai-skills/skills/image-skills/SKILL.md
```

Expected:
```
## 환경 설정
## § 1. Imagen 4 이미지 생성
## § 2. Gemini 이미지 생성
## § 3. Gemini 이미지 편집
## § 4. Best Practices
## § 5. Limitations
## § 6. Responsible AI & Safety
```

- [ ] **Step 3: Commit**

```bash
cd /home/ext_hajekim_google_com/sandbox/vertexai-skills
git add skills/image-skills/SKILL.md
git commit -m "feat: add §5 limitations and §6 responsible AI to image-skills"
```

---

## Task 7: image-skills — 심볼릭 링크 생성

**Files:**
- Symlink: `~/.gemini/skills/image-skills` → `skills/image-skills/`

- [ ] **Step 1: 심볼릭 링크 생성**

```bash
ln -s /home/ext_hajekim_google_com/sandbox/vertexai-skills/skills/image-skills \
      /home/ext_hajekim_google_com/.gemini/skills/image-skills
```

- [ ] **Step 2: 링크 및 접근 확인**

```bash
ls -la /home/ext_hajekim_google_com/.gemini/skills/ | grep image-skills
head -5 /home/ext_hajekim_google_com/.gemini/skills/image-skills/SKILL.md
```

Expected:
```
image-skills -> /home/ext_hajekim_google_com/sandbox/vertexai-skills/skills/image-skills
---
name: image-skills
```

- [ ] **Step 3: Commit**

```bash
cd /home/ext_hajekim_google_com/sandbox/vertexai-skills
git add .
git commit -m "feat: deploy image-skills via symlink to ~/.gemini/skills"
```

---

# PART B: video-skills

---

## Task 8: video-skills — frontmatter + 환경 설정

**Files:**
- Create: `skills/video-skills/SKILL.md`

- [ ] **Step 1: 디렉토리 생성 및 SKILL.md 작성**

```bash
mkdir -p /home/ext_hajekim_google_com/sandbox/vertexai-skills/skills/video-skills
```

`skills/video-skills/SKILL.md`를 아래 내용으로 생성:

````markdown
---
name: video-skills
description: Use when working with video generation on Vertex AI using google-genai Python SDK — imports Veo models, or requests involving "video generation", "generate video", "text to video", "image to video", "extend video", "Veo". 한국어: "비디오 생성", "영상 만들기", "텍스트로 영상", "이미지로 영상", "비오", "영상 연장".
version: 1.0.0
---

# Video Skills — google-genai Python SDK 레퍼런스

## 환경 설정

### 라이브러리 설치

```bash
pip install --upgrade google-genai
```

### 필수 환경변수 및 GCS 버킷

```bash
export GOOGLE_CLOUD_PROJECT="your-project-id"
export GOOGLE_CLOUD_LOCATION="us-central1"
export GOOGLE_GENAI_USE_VERTEXAI="True"
# 생성된 비디오는 GCS 버킷에 저장됨 — 필수
export OUTPUT_GCS_URI="gs://your-bucket/output/"
```

### 클라이언트 초기화

```python
from google import genai

client = genai.Client()
output_gcs_uri = "gs://your-bucket/output/"  # 모든 생성 작업에 필요
```

### 모델 목록

| 모델 | 특징 |
|------|------|
| `veo-3.1-generate-001` | 최신 안정 버전 (권장) |
| `veo-3.1-fast-generate-001` | 빠른 생성 |
| `veo-3.0-generate-001` | 이전 안정 버전 |
| `veo-2.0-generate-001` | 레거시 (객체 삽입/제거 전용) |

### 공통 패턴: 폴링 루프

모든 비디오 생성은 비동기 LRO(Long Running Operation)로 동작한다:

```python
import time

# 작업 시작 후 완료까지 대기
while not operation.done:
    time.sleep(15)
    operation = client.operations.get(operation)

for video in operation.result.generated_videos:
    print(video.video.uri)  # GCS URI
```

> **원칙:** 비디오 생성은 항상 비동기다. `operation.done`이 True가 될 때까지 15초 간격으로 폴링한다.
````

- [ ] **Step 2: 파일 존재 확인**

```bash
head -20 /home/ext_hajekim_google_com/sandbox/vertexai-skills/skills/video-skills/SKILL.md
```

Expected: frontmatter와 환경 설정 섹션 출력.

- [ ] **Step 3: Commit**

```bash
cd /home/ext_hajekim_google_com/sandbox/vertexai-skills
git add skills/video-skills/SKILL.md
git commit -m "feat: scaffold video-skills SKILL.md with env setup"
```

---

## Task 9: video-skills — § 1 텍스트 → 비디오

**Files:**
- Modify: `skills/video-skills/SKILL.md`

- [ ] **Step 1: § 1 섹션 추가**

````markdown

---

## § 1. 텍스트 → 비디오

### 언제: 텍스트 프롬프트만으로 비디오를 생성할 때

```python
import time
from google import genai
from google.genai.types import GenerateVideosConfig

client = genai.Client()
output_gcs_uri = "gs://your-bucket/output/"

operation = client.models.generate_videos(
    model="veo-3.1-generate-001",
    prompt="A cat reading a book in a cozy library, warm lighting, cinematic shot",
    config=GenerateVideosConfig(
        aspect_ratio="16:9",       # "16:9" (가로) 또는 "9:16" (세로/모바일)
        number_of_videos=1,        # 생성할 비디오 수 (1-4)
        duration_seconds=8,        # 비디오 길이 (4, 6, 8 중 선택)
        output_gcs_uri=output_gcs_uri,
    ),
)

# 완료까지 대기
while not operation.done:
    time.sleep(15)
    operation = client.operations.get(operation)

for video in operation.result.generated_videos:
    print(video.video.uri)   # gs://your-bucket/output/xxx.mp4
```

### 파라미터 가이드

| 파라미터 | 값 | 설명 |
|---------|-----|------|
| `aspect_ratio` | `"16:9"` / `"9:16"` | 가로형 / 세로형(모바일) |
| `number_of_videos` | 1-4 | 생성할 비디오 수 |
| `duration_seconds` | 4, 6, 8 | 길이 (초) |

> **원칙:**
> - `output_gcs_uri`는 필수다. 비디오는 로컬이 아닌 GCS에 저장된다.
> - 생성 시간은 보통 2-5분 소요된다.
````

- [ ] **Step 2: Commit**

```bash
cd /home/ext_hajekim_google_com/sandbox/vertexai-skills
git add skills/video-skills/SKILL.md
git commit -m "feat: add §1 text-to-video to video-skills"
```

---

## Task 10: video-skills — § 2 이미지 → 비디오

**Files:**
- Modify: `skills/video-skills/SKILL.md`

- [ ] **Step 1: § 2 섹션 추가**

````markdown

---

## § 2. 이미지 → 비디오

### 언제: 정적 이미지에 움직임을 추가하여 비디오로 만들 때

```python
import time
from google import genai
from google.genai.types import GenerateVideosConfig, Image

client = genai.Client()
output_gcs_uri = "gs://your-bucket/output/"

operation = client.models.generate_videos(
    model="veo-3.1-generate-001",
    prompt="The flowers sway gently in the breeze, petals trembling",
    image=Image(
        gcs_uri="gs://your-bucket/input/flowers.png",   # GCS 경로 필수
        mime_type="image/png",                           # "image/png" 또는 "image/jpeg"
    ),
    config=GenerateVideosConfig(
        aspect_ratio="16:9",
        output_gcs_uri=output_gcs_uri,
    ),
)

while not operation.done:
    time.sleep(15)
    operation = client.operations.get(operation)

for video in operation.result.generated_videos:
    print(video.video.uri)
```

> **원칙:**
> - 입력 이미지는 반드시 GCS에 업로드 후 `gcs_uri`로 참조한다.
> - 권장 해상도: 720p 이상.
> - 프롬프트는 이미지에 추가할 **동작(motion)**만 설명한다. 이미지 내용을 재설명하지 않는다.
````

- [ ] **Step 2: Commit**

```bash
cd /home/ext_hajekim_google_com/sandbox/vertexai-skills
git add skills/video-skills/SKILL.md
git commit -m "feat: add §2 image-to-video to video-skills"
```

---

## Task 11: video-skills — § 3 첫/마지막 프레임으로 비디오

**Files:**
- Modify: `skills/video-skills/SKILL.md`

- [ ] **Step 1: § 3 섹션 추가**

````markdown

---

## § 3. 첫/마지막 프레임으로 비디오

### 언제: 시작 이미지와 끝 이미지 사이의 전환을 자동 생성할 때

```python
import time
from google import genai
from google.genai.types import GenerateVideosConfig, Image

client = genai.Client()
output_gcs_uri = "gs://your-bucket/output/"

operation = client.models.generate_videos(
    model="veo-3.1-generate-001",
    prompt="A smooth natural transition, cinematic feel",
    image=Image(
        gcs_uri="gs://your-bucket/first-frame.png",   # 시작 프레임
        mime_type="image/png",
    ),
    config=GenerateVideosConfig(
        aspect_ratio="16:9",
        last_frame=Image(
            gcs_uri="gs://your-bucket/last-frame.png",   # 마지막 프레임
            mime_type="image/png",
        ),
        output_gcs_uri=output_gcs_uri,
    ),
)

while not operation.done:
    time.sleep(15)
    operation = client.operations.get(operation)

for video in operation.result.generated_videos:
    print(video.video.uri)
```

> **원칙:**
> - `image` (첫 프레임)은 `generate_videos()` 직접 파라미터로 전달한다.
> - `last_frame` (마지막 프레임)은 `GenerateVideosConfig` 안에 전달한다.
> - 두 이미지의 해상도와 비율을 일치시킨다.
````

- [ ] **Step 2: Commit**

```bash
cd /home/ext_hajekim_google_com/sandbox/vertexai-skills
git add skills/video-skills/SKILL.md
git commit -m "feat: add §3 first/last frame video to video-skills"
```

---

## Task 12: video-skills — § 4 비디오 연장

**Files:**
- Modify: `skills/video-skills/SKILL.md`

- [ ] **Step 1: § 4 섹션 추가**

````markdown

---

## § 4. 비디오 연장 (Extend)

### 언제: 기존 비디오를 자연스럽게 이어서 늘릴 때

```python
import time
from google import genai
from google.genai.types import GenerateVideosConfig, Video

client = genai.Client()
output_gcs_uri = "gs://your-bucket/output/"

operation = client.models.generate_videos(
    model="veo-3.1-generate-001",
    prompt="Continue the scene naturally, same lighting and style",
    video=Video(
        uri="gs://your-bucket/input/original.mp4",   # 원본 비디오 GCS 경로
        mime_type="video/mp4",
    ),
    config=GenerateVideosConfig(
        output_gcs_uri=output_gcs_uri,
    ),
)

while not operation.done:
    time.sleep(15)
    operation = client.operations.get(operation)

for video in operation.result.generated_videos:
    print(video.video.uri)
```

### 입력 비디오 요구사항

| 항목 | 조건 |
|------|------|
| 포맷 | MP4 전용 |
| 길이 | 1-30초 |
| 프레임레이트 | 24fps |
| 해상도 | 720p, 1080p, 4K |
| 비율 | 9:16 또는 16:9 |

### 출력 사양

- 포맷: MP4
- 연장 길이: 7초
- 해상도/비율: 입력과 동일 유지

> **원칙:**
> - `Video` 객체에 `uri`(GCS)와 `mime_type="video/mp4"`를 반드시 명시한다.
> - 프롬프트는 원본 스타일을 유지한다는 것을 명시한다.
````

- [ ] **Step 2: Commit**

```bash
cd /home/ext_hajekim_google_com/sandbox/vertexai-skills
git add skills/video-skills/SKILL.md
git commit -m "feat: add §4 video extension to video-skills"
```

---

## Task 13: video-skills — § 5 고급 기법

**Files:**
- Modify: `skills/video-skills/SKILL.md`

- [ ] **Step 1: § 5 섹션 추가**

````markdown

---

## § 5. 고급 기법

### 5-1. 레퍼런스 이미지로 스타일/캐릭터 가이드

인물, 캐릭터, 제품의 외관을 일관되게 유지하거나 특정 스타일을 적용할 때:

```python
import time
from google import genai
from google.genai.types import (
    GenerateVideosConfig,
    Image,
    VideoGenerationReferenceImage,
)

client = genai.Client()
output_gcs_uri = "gs://your-bucket/output/"

operation = client.models.generate_videos(
    model="veo-3.1-generate-001",
    prompt="A person walking through a sunny park, cheerful mood",
    config=GenerateVideosConfig(
        reference_images=[
            VideoGenerationReferenceImage(
                image=Image(
                    gcs_uri="gs://your-bucket/person.png",
                    mime_type="image/png",
                ),
                reference_type="asset",   # "asset": 인물/캐릭터/제품 외관 유지
            ),
        ],
        aspect_ratio="16:9",
        output_gcs_uri=output_gcs_uri,
    ),
)

while not operation.done:
    time.sleep(15)
    operation = client.operations.get(operation)
```

| `reference_type` | 용도 | 지원 모델 |
|-----------------|------|---------|
| `"asset"` | 인물/캐릭터/제품 외관 유지 | veo-3.1 |
| `"style"` | 영상 아트 스타일 적용 | veo-2.0 전용 |

### 5-2. 객체 삽입 (Insert Objects)

> **주의:** `veo-2.0-generate-001` 전용 기능

```python
from google.genai.types import GenerateVideosConfig, Video, Image, VideoGenerationMask, VideoGenerationMaskMode

operation = client.models.generate_videos(
    model="veo-2.0-generate-001",
    prompt="a sheep standing in the field",
    video=Video(
        uri="gs://your-bucket/input.mp4",
        mime_type="video/mp4",
    ),
    config=GenerateVideosConfig(
        mask=VideoGenerationMask(
            image=Image(
                gcs_uri="gs://your-bucket/mask.png",   # 삽입 위치 마스크
                mime_type="image/png",
            ),
            mask_mode=VideoGenerationMaskMode.INSERT,
        ),
        output_gcs_uri=output_gcs_uri,
    ),
)
```

### 5-3. 객체 제거 (Remove Objects)

> **주의:** `veo-2.0-generate-001` 전용 기능

```python
operation = client.models.generate_videos(
    model="veo-2.0-generate-001",
    prompt="clean background without the object",
    video=Video(
        uri="gs://your-bucket/input.mp4",
        mime_type="video/mp4",
    ),
    config=GenerateVideosConfig(
        mask=VideoGenerationMask(
            image=Image(
                gcs_uri="gs://your-bucket/mask.png",   # 제거 대상 마스크
                mime_type="image/png",
            ),
            mask_mode=VideoGenerationMaskMode.REMOVE,
        ),
        output_gcs_uri=output_gcs_uri,
    ),
)
```

> **원칙:**
> - 레퍼런스 이미지는 최대 3개 까지 제공 가능 (asset 타입).
> - 객체 삽입/제거는 `veo-2.0-generate-001`에서만 지원된다.
> - 마스크 이미지는 PNG/JPEG/WebP 형식이며, 대상 영역을 흰색으로 표시한다.
````

- [ ] **Step 2: Commit**

```bash
cd /home/ext_hajekim_google_com/sandbox/vertexai-skills
git add skills/video-skills/SKILL.md
git commit -m "feat: add §5 advanced techniques to video-skills"
```

---

## Task 14: video-skills — § 6 프롬프트 가이드 & Best Practices

**Files:**
- Modify: `skills/video-skills/SKILL.md`

- [ ] **Step 1: § 6 섹션 추가**

````markdown

---

## § 6. 프롬프트 가이드 & Best Practices

### 프롬프트 7가지 구성 요소

| 요소 | 설명 | 예시 |
|------|------|------|
| **Subject** | 주인공/대상 | `"a seasoned detective"`, `"a golden retriever"` |
| **Action** | 동작/움직임 | `"walking slowly"`, `"leaves rustling"` |
| **Scene** | 배경/환경 | `"foggy Victorian street at night"` |
| **Camera angle** | 카메라 각도 | `"low-angle shot"`, `"bird's-eye view"`, `"Dutch angle"` |
| **Camera movement** | 카메라 움직임 | `"slow pan left"`, `"dolly zoom"`, `"aerial drone shot"` |
| **Visual style** | 시각 스타일 | `"photorealistic"`, `"anime style"`, `"cinematic lighting"` |
| **Temporal** | 시간 효과 | `"slow motion"`, `"time-lapse"`, `"fast-paced"` |

### 프롬프트 작성 원칙

**구체적이고 명확하게:**
```
# 나쁜 예
"a man looking sad"

# 좋은 예
"Low-angle close-up shot of a man with a somber expression, rain falling in background, cinematic lighting"
```

**대화/발화 표현:**
```
# 나쁜 예 (따옴표 사용)
'A woman saying "My name is Clara"'

# 좋은 예 (콜론 사용)
"A woman says: My name is Clara"
```

**한 씬에 집중:**
- 여러 장면을 한 프롬프트에 담지 않는다
- 각 비디오는 단일 장면으로 제한한다

**이미지→비디오 시:**
- 소스 이미지 내용을 재설명하지 않는다
- 추가할 **동작(motion)**만 설명한다

### 프롬프트 리라이터 끄기

> **주의:** `veo-2.0-generate-001`에서만 지원. Veo 3/3.1은 비활성화 불가.

```python
config=GenerateVideosConfig(
    enhance_prompt=False,    # 기본값 True (모델이 프롬프트를 자동 개선)
    output_gcs_uri=output_gcs_uri,
)
```

리라이터를 끄면 출력 품질과 프롬프트 반영도가 낮아질 수 있다.

### Gemini로 프롬프트 개선하기

```python
from google import genai
from google.genai.types import GenerateContentConfig

text_client = genai.Client()

refined = text_client.models.generate_content(
    model="gemini-2.5-flash",
    contents="Refine this video prompt for Veo: 'a cat in a garden'",
    config=GenerateContentConfig(
        system_instruction="You are a Veo video prompt expert. Make prompts cinematic, specific, and detailed. Keep under 200 words.",
    ),
)
print(refined.text)  # 개선된 프롬프트를 veo에 전달
```
````

- [ ] **Step 2: Commit**

```bash
cd /home/ext_hajekim_google_com/sandbox/vertexai-skills
git add skills/video-skills/SKILL.md
git commit -m "feat: add §6 prompt guide and best practices to video-skills"
```

---

## Task 15: video-skills — § 7 Responsible AI & Safety

**Files:**
- Modify: `skills/video-skills/SKILL.md`

- [ ] **Step 1: § 7 섹션 추가 및 전체 구조 검증**

````markdown

---

## § 7. Responsible AI & Safety

### 금지 콘텐츠

Generative AI Prohibited Use Policy에 의해 금지:
- 아동 안전 위협 콘텐츠
- 폭력적/극단적 콘텐츠
- 성적으로 노골적인 콘텐츠
- 혐오 발언 / 독성 언어
- 유명인/실존 인물의 무단 사실적 표현

### Safety 필터 에러 처리

```python
import time
from google import genai
from google.genai.types import GenerateVideosConfig

client = genai.Client()
output_gcs_uri = "gs://your-bucket/output/"

operation = client.models.generate_videos(
    model="veo-3.1-generate-001",
    prompt="Your video prompt here",
    config=GenerateVideosConfig(output_gcs_uri=output_gcs_uri),
)

while not operation.done:
    time.sleep(15)
    operation = client.operations.get(operation)

# 결과 확인 및 Safety 에러 처리
if operation.result and operation.result.generated_videos:
    for video in operation.result.generated_videos:
        print(video.video.uri)
elif operation.error:
    print(f"Error: {operation.error.message}")
    # Safety 관련 에러 코드 예시:
    # 58061214 / 17301594 → 아동 안전
    # 90789179 / 43188360 → 성적 콘텐츠
    # 61493863 / 56562880 → 폭력
    # 57734940 / 22137204 → 혐오 발언
    # 29310472 / 15236754 → 유명인 묘사
```

> **원칙:**
> - Safety 필터로 차단된 경우 프롬프트를 수정한다.
> - 일부 비디오만 차단되고 나머지는 반환될 수 있다 (partial results).
> - 유명인/실존 인물의 사실적 표현은 별도 승인이 필요하다.
````

- [ ] **Step 2: 전체 섹션 구조 확인**

```bash
grep "^## " /home/ext_hajekim_google_com/sandbox/vertexai-skills/skills/video-skills/SKILL.md
```

Expected:
```
## 환경 설정
## § 1. 텍스트 → 비디오
## § 2. 이미지 → 비디오
## § 3. 첫/마지막 프레임으로 비디오
## § 4. 비디오 연장 (Extend)
## § 5. 고급 기법
## § 6. 프롬프트 가이드 & Best Practices
## § 7. Responsible AI & Safety
```

- [ ] **Step 3: Commit**

```bash
cd /home/ext_hajekim_google_com/sandbox/vertexai-skills
git add skills/video-skills/SKILL.md
git commit -m "feat: add §7 responsible AI and safety to video-skills"
```

---

## Task 16: video-skills — 심볼릭 링크 생성 및 최종 검증

**Files:**
- Symlink: `~/.gemini/skills/video-skills` → `skills/video-skills/`

- [ ] **Step 1: 심볼릭 링크 생성**

```bash
ln -s /home/ext_hajekim_google_com/sandbox/vertexai-skills/skills/video-skills \
      /home/ext_hajekim_google_com/.gemini/skills/video-skills
```

- [ ] **Step 2: 두 스킬 모두 최종 확인**

```bash
ls -la /home/ext_hajekim_google_com/.gemini/skills/ | grep -E "image|video"
echo "--- image-skills ---"
grep "^## " /home/ext_hajekim_google_com/.gemini/skills/image-skills/SKILL.md
echo "--- video-skills ---"
grep "^## " /home/ext_hajekim_google_com/.gemini/skills/video-skills/SKILL.md
```

Expected:
```
image-skills -> .../skills/image-skills
video-skills -> .../skills/video-skills
--- image-skills ---
## 환경 설정
## § 1. Imagen 4 이미지 생성
## § 2. Gemini 이미지 생성
## § 3. Gemini 이미지 편집
## § 4. Best Practices
## § 5. Limitations
## § 6. Responsible AI & Safety
--- video-skills ---
## 환경 설정
## § 1. 텍스트 → 비디오
## § 2. 이미지 → 비디오
## § 3. 첫/마지막 프레임으로 비디오
## § 4. 비디오 연장 (Extend)
## § 5. 고급 기법
## § 6. 프롬프트 가이드 & Best Practices
## § 7. Responsible AI & Safety
```

- [ ] **Step 3: git log 확인**

```bash
cd /home/ext_hajekim_google_com/sandbox/vertexai-skills && git log --oneline | head -20
```

Expected: 16개 이상의 커밋 확인.

- [ ] **Step 4: 최종 Commit**

```bash
cd /home/ext_hajekim_google_com/sandbox/vertexai-skills
git add .
git commit -m "feat: deploy video-skills via symlink to ~/.gemini/skills"
```
