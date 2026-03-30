---
name: grounding-skills
description: Use when working with grounding on Vertex AI using google-genai Python SDK — imports GoogleSearch, GoogleMaps, VertexAISearch, Elasticsearch, ExternalApi, EnterpriseWebSearch, or requests involving "grounding", "Google Search grounding", "RAG", "Vertex AI Search", "Elasticsearch grounding", "enterprise web search". 한국어: "그라운딩", "검색 기반 생성", "구글 검색 연동", "버텍스 검색", "환각 방지", "외부 데이터 연결".
version: 1.0.0
---

# Grounding Skills — google-genai Python SDK 레퍼런스

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

### 그라운딩 방식 선택 가이드

| 방식 | Tool 클래스 | 용도 |
|------|------------|------|
| Google Search | `GoogleSearch` | 최신 공개 웹 데이터 |
| Google Maps | `GoogleMaps` | 장소/지리 정보 |
| Vertex AI Search | `VertexAISearch` (via `Retrieval`) | 내 문서/웹사이트 데이터 |
| Elasticsearch | `Elasticsearch` (via `Retrieval`) | 기존 ES 인덱스 |
| Custom Search API | `ExternalApi` (via `Retrieval`) | 자체 검색 API |
| Parallel Web Search | REST only (`parallelAiSearch`) | LLM 최적화 웹 인덱스 |
| Enterprise Web Search | `EnterpriseWebSearch` | 컴플라이언스 규제 환경 |

> **참고:** Vertex AI RAG Engine 그라운딩(`ground-responses-using-rag`)은 `vertexai` SDK(`google-cloud-aiplatform`)를 사용하며, 이 스킬의 `google-genai` SDK 범위에 포함되지 않는다.

---

## § 1. Google Search 그라운딩

### 언제: 최신 공개 웹 정보로 응답을 보강할 때

```python
from google import genai
from google.genai.types import (
    GenerateContentConfig,
    GoogleSearch,
    HttpOptions,
    Tool,
)

client = genai.Client(http_options=HttpOptions(api_version="v1"))

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="When is the next total solar eclipse in the United States?",
    config=GenerateContentConfig(
        tools=[
            Tool(
                google_search=GoogleSearch(
                    exclude_domains=["domain.com"],   # 제외할 도메인 (선택)
                )
            )
        ],
    ),
)

print(response.text)
# 검색 출처 확인
for chunk in response.candidates[0].grounding_metadata.grounding_chunks:
    print(chunk.web.uri)
```

### Dynamic Retrieval (조건부 그라운딩)

```python
from google.genai.types import DynamicRetrievalConfig, GoogleSearch, Tool

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="What is the capital of France?",
    config=GenerateContentConfig(
        tools=[
            Tool(
                google_search=GoogleSearch(
                    dynamic_retrieval_config=DynamicRetrievalConfig(
                        dynamic_threshold=0.6,   # 0.0~1.0: 낮을수록 더 자주 검색
                    )
                )
            )
        ],
    ),
)
```

### GoogleSearch 파라미터

| 파라미터 | 타입 | 설명 |
|---------|------|------|
| `exclude_domains` | `list[str]` | 그라운딩에서 제외할 도메인 목록 |
| `dynamic_retrieval_config` | `DynamicRetrievalConfig` | 조건부 그라운딩 설정 |
| `dynamic_threshold` | `float` (0.0~1.0) | 검색 발동 임계값 |

> **원칙:**
> - 권장 temperature: `1.0`.
> - 일일 최대 1,000,000 쿼리 제한.
> - Search Suggestion HTML/CSS(`searchEntryPoint`)는 수정 없이 그대로 표시해야 한다.
