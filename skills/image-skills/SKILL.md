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
