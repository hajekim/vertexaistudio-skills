---
name: thinking-skills
description: Use when working with thinking/reasoning models on Vertex AI using google-genai Python SDK — imports ThinkingConfig, or requests involving "thinking budget", "thinking level", "thought signatures", "reasoning model", "extended thinking". 한국어: "사고 모드", "추론 모델", "생각 예산", "사고 시그니처", "씽킹".
version: 1.0.0
---

# Thinking Skills — google-genai Python SDK 레퍼런스

## 환경 설정

### 라이브러리 설치

```bash
pip install --upgrade google-genai
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
from google.genai.types import HttpOptions

client = genai.Client(http_options=HttpOptions(api_version="v1"))
```

### 모델 선택 가이드

| 모델 | 사고 방식 | 비활성화 가능 | 최대 토큰 |
|------|----------|-------------|---------|
| `gemini-2.5-flash` | `thinking_budget` | ✅ (0으로 설정) | 24,576 |
| `gemini-2.5-flash-lite` | `thinking_budget` | ✅ (0으로 설정) | 24,576 |
| `gemini-2.5-pro` | `thinking_budget` | ❌ | 32,768 |
| `gemini-3-flash` | `thinking_level` | ✅ | — |
| `gemini-3.1-pro` | `thinking_level` | ❌ | — |

> **선택 기준:**
> - Gemini 2.5 → `thinking_budget`으로 세밀한 토큰 예산 제어
> - Gemini 3+ → `thinking_level`로 사고 강도 선택
> - `thinking_budget`과 `thinking_level`을 동시에 지정할 수 없다.
