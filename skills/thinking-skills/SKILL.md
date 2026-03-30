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

---

## § 1. Thinking Budget 설정 (Gemini 2.5)

### 언제: 토큰 예산으로 사고 깊이를 직접 제어할 때

```python
from google import genai
from google.genai.types import GenerateContentConfig, HttpOptions, ThinkingConfig

client = genai.Client(http_options=HttpOptions(api_version="v1"))

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="solve x^2 + 4x + 4 = 0",
    config=GenerateContentConfig(
        thinking_config=ThinkingConfig(
            thinking_budget=1024,   # 사고에 사용할 최대 토큰 수
        ),
    ),
)

print(response.text)
# 사고 토큰 사용량 확인
print(f"Thoughts: {response.usage_metadata.thoughts_token_count}")
print(f"Total:    {response.usage_metadata.total_token_count}")
```

### thinking_budget 값 가이드

| 값 | 동작 |
|----|------|
| `1` ~ 모델 최대값 | 지정한 토큰 수 내에서 사고 |
| `0` | 사고 비활성화 (gemini-2.5-pro는 지원 안 함) |
| `-1` | 동적 자동 제어 (기본값, 최대 8,192) |

### 모델별 thinking_budget 범위

| 모델 | 최솟값 | 최댓값 |
|------|--------|--------|
| `gemini-2.5-flash` | 1 | 24,576 |
| `gemini-2.5-flash-lite` | 512 | 24,576 |
| `gemini-2.5-pro` | 128 | 32,768 |

> **원칙:**
> - `thinking_budget=0`으로 사고를 비활성화하면 응답 속도가 빨라지지만 추론 품질이 낮아질 수 있다.
> - `gemini-2.5-pro`는 `thinking_budget=0`을 지원하지 않는다.
> - 파인튜닝은 사고 활성화 상태에서 지원하지 않는다.
