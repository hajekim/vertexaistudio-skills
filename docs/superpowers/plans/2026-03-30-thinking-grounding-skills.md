# Thinking & Grounding Skills Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** `thinking-skills/SKILL.md`과 `grounding-skills/SKILL.md` 두 Gemini CLI 스킬 파일을 작성하고 `~/.gemini/skills/`에 심볼릭 링크로 배포.

**Architecture:** 기존 프로젝트 규칙 준수. `google-genai` SDK 전용, `HttpOptions(api_version="v1")` 표준 초기화, 심볼릭 링크 배포.

**Tech Stack:** `google-genai` Python SDK, Gemini CLI skills 시스템 (SKILL.md frontmatter)

---

## File Map

| 파일 | 역할 |
|------|------|
| `skills/thinking-skills/SKILL.md` | 사고 모델 제어 스킬 소스 |
| `skills/grounding-skills/SKILL.md` | 그라운딩 스킬 소스 |
| `~/.gemini/skills/thinking-skills` → `skills/thinking-skills/` | Gemini CLI 배포 링크 |
| `~/.gemini/skills/grounding-skills` → `skills/grounding-skills/` | Gemini CLI 배포 링크 |

---

# PART A: thinking-skills

---

## Task A-1: thinking-skills — frontmatter + 환경 설정

**Files:**
- Create: `skills/thinking-skills/SKILL.md`

- [ ] **Step 1: 디렉토리 생성 및 SKILL.md 작성**

```bash
mkdir -p /home/ext_hajekim_google_com/sandbox/vertexai-skills/skills/thinking-skills
```

`skills/thinking-skills/SKILL.md`를 아래 내용으로 생성:

````markdown
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
````

- [ ] **Step 2: 파일 확인**

```bash
head -20 /home/ext_hajekim_google_com/sandbox/vertexai-skills/skills/thinking-skills/SKILL.md
```

- [ ] **Step 3: Commit**

```bash
cd /home/ext_hajekim_google_com/sandbox/vertexai-skills
git add skills/thinking-skills/SKILL.md
git commit -m "feat: scaffold thinking-skills SKILL.md with env setup"
```

---

## Task A-2: thinking-skills — § 1 Thinking Budget 설정

**Files:**
- Modify: `skills/thinking-skills/SKILL.md`

- [ ] **Step 1: § 1 섹션 추가**

````markdown

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
````

- [ ] **Step 2: Commit**

```bash
cd /home/ext_hajekim_google_com/sandbox/vertexai-skills
git add skills/thinking-skills/SKILL.md
git commit -m "feat: add §1 thinking budget to thinking-skills"
```

---

## Task A-3: thinking-skills — § 2 Thinking Level 설정

**Files:**
- Modify: `skills/thinking-skills/SKILL.md`

- [ ] **Step 1: § 2 섹션 추가**

````markdown

---

## § 2. Thinking Level 설정 (Gemini 3+)

### 언제: Gemini 3 이상 모델에서 사고 강도를 선택할 때

```python
from google import genai
from google.genai import types
from google.genai.types import HttpOptions

client = genai.Client(http_options=HttpOptions(api_version="v1"))

response = client.models.generate_content(
    model="gemini-3.1-pro",
    contents="How does AI work?",
    config=types.GenerateContentConfig(
        thinking_config=types.ThinkingConfig(
            thinking_level=types.ThinkingLevel.HIGH,
        ),
    ),
)

print(response.text)
```

### ThinkingLevel 열거형 값

| 값 | 토큰 사용 | 적합한 태스크 | 지원 모델 |
|----|---------|------------|---------|
| `ThinkingLevel.MINIMAL` | 최소 | 간단한 분류, 단순 답변 | gemini-3-flash, gemini-3.1-flash-lite |
| `ThinkingLevel.LOW` | 적음 | 일반 질답 | 모든 Gemini 3 |
| `ThinkingLevel.MEDIUM` | 보통 | 중간 복잡도 문제 | 모든 Gemini 3 |
| `ThinkingLevel.HIGH` | 많음 | 복잡한 추론, 수학, 코드 | 모든 Gemini 3 |

> **원칙:**
> - `ThinkingLevel.MINIMAL`은 Thought Signatures가 필요하며, 없으면 400 오류가 발생한다.
> - `thinking_level`과 `thinking_budget`을 동시에 지정할 수 없다.
> - `gemini-3.1-pro`는 사고를 비활성화할 수 없다.
````

- [ ] **Step 2: Commit**

```bash
cd /home/ext_hajekim_google_com/sandbox/vertexai-skills
git add skills/thinking-skills/SKILL.md
git commit -m "feat: add §2 thinking level to thinking-skills"
```

---

## Task A-4: thinking-skills — § 3 Thought 요약 보기

**Files:**
- Modify: `skills/thinking-skills/SKILL.md`

- [ ] **Step 1: § 3 섹션 추가**

````markdown

---

## § 3. Thought 요약 보기

### 언제: 모델의 사고 과정을 응답과 함께 확인할 때

```python
from google import genai
from google.genai.types import GenerateContentConfig, HttpOptions, ThinkingConfig

client = genai.Client(http_options=HttpOptions(api_version="v1"))

response = client.models.generate_content(
    model="gemini-2.5-pro",
    contents="solve x^2 + 4x + 4 = 0",
    config=GenerateContentConfig(
        thinking_config=ThinkingConfig(
            include_thoughts=True,   # 사고 요약 포함
        ),
    ),
)

# 최종 답변과 사고 과정 분리
for part in response.candidates[0].content.parts:
    if part.thought:
        print(f"[Thought] {part.text}")
    else:
        print(f"[Answer]  {part.text}")
```

> **원칙:**
> - `include_thoughts=True`는 베스트 에포트(best-effort) 기능이다 — 항상 사고 내용이 반환되지 않을 수 있다.
> - `part.thought`가 `True`인 파트가 사고 요약, `False`인 파트가 최종 응답이다.
> - 사고 요약은 축약본이다. 완전한 내부 사고 과정이 아니다.
````

- [ ] **Step 2: Commit**

```bash
cd /home/ext_hajekim_google_com/sandbox/vertexai-skills
git add skills/thinking-skills/SKILL.md
git commit -m "feat: add §3 thought summaries to thinking-skills"
```

---

## Task A-5: thinking-skills — § 4 Thought Signatures

**Files:**
- Modify: `skills/thinking-skills/SKILL.md`

- [ ] **Step 1: § 4 섹션 추가**

````markdown

---

## § 4. Thought Signatures (함수 호출 시)

### 언제: 함수 호출이 포함된 멀티턴 대화에서 사고 연속성을 유지할 때

Thought Signatures는 함수 호출 사이에 모델의 사고 상태를 유지하는 암호화된 토큰이다. SDK를 사용하면 자동으로 처리된다.

```python
from google import genai
from google.genai.types import (
    Content,
    FunctionDeclaration,
    GenerateContentConfig,
    HttpOptions,
    Part,
    ThinkingConfig,
    Tool,
)

client = genai.Client(http_options=HttpOptions(api_version="v1"))

# 1. 도구 정의
get_weather = FunctionDeclaration(
    name="get_weather",
    description="Gets current weather for a location.",
    parameters={
        "type": "object",
        "properties": {"location": {"type": "string"}},
        "required": ["location"],
    },
)
tool = Tool(function_declarations=[get_weather])

# 2. 첫 번째 요청 — 모델이 함수 호출 요청
prompt = "What's the weather in Seoul?"
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=prompt,
    config=GenerateContentConfig(
        tools=[tool],
        thinking_config=ThinkingConfig(include_thoughts=True),
    ),
)

# 3. 함수 실행 (실제 API 호출 또는 목 데이터)
func_call = response.function_calls[0]
func_result = {"location": func_call.args["location"], "temperature": "18°C"}

# 4. 함수 결과 포함 히스토리 구성 — 서명은 SDK가 자동 처리
history = [
    Content(role="user", parts=[Part(text=prompt)]),
    response.candidates[0].content,   # Thought Signature 포함
    Content(
        role="tool",
        parts=[Part.from_function_response(name=func_call.name, response=func_result)],
    ),
]

# 5. 두 번째 요청 — 서명 덕분에 사고 연속성 유지
response2 = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=history,
    config=GenerateContentConfig(
        tools=[tool],
        thinking_config=ThinkingConfig(include_thoughts=True),
    ),
)

print(response2.text)
```

### Thought Signature 규칙

| 규칙 | 설명 |
|------|------|
| SDK 자동 처리 | `chat` API나 `response.candidates[0].content` 그대로 전달하면 자동 포함 |
| Part 수정 금지 | 서명이 있는 `Part`를 다른 `Part`와 합치거나 수정하면 안 된다 |
| 병렬 함수 호출 | 첫 번째 `functionCall` Part에만 서명 포함, 모든 함수 호출은 응답보다 먼저 배치 |
| Gemini 3 필수 | Gemini 3+ 모델은 서명 없이 요청 시 400 오류 반환 |

> **원칙:**
> - `chat.send_message()`를 사용하면 서명이 자동으로 처리된다.
> - REST API로 직접 구현 시에는 `Part` 단위로 서명을 보존해야 한다.
> - 서명 검증을 건너뛰려면 `skip_thought_signature_validator`를 사용할 수 있지만 성능이 저하된다.
````

- [ ] **Step 2: 전체 섹션 구조 확인**

```bash
grep "^## " /home/ext_hajekim_google_com/sandbox/vertexai-skills/skills/thinking-skills/SKILL.md
```

Expected:
```
## 환경 설정
## § 1. Thinking Budget 설정 (Gemini 2.5)
## § 2. Thinking Level 설정 (Gemini 3+)
## § 3. Thought 요약 보기
## § 4. Thought Signatures (함수 호출 시)
```

- [ ] **Step 3: Commit**

```bash
cd /home/ext_hajekim_google_com/sandbox/vertexai-skills
git add skills/thinking-skills/SKILL.md
git commit -m "feat: add §4 thought signatures to thinking-skills"
```

---

## Task A-6: thinking-skills — 심볼릭 링크 생성

- [ ] **Step 1: 심볼릭 링크 생성**

```bash
ln -s /home/ext_hajekim_google_com/sandbox/vertexai-skills/skills/thinking-skills \
      /home/ext_hajekim_google_com/.gemini/skills/thinking-skills
```

- [ ] **Step 2: 확인**

```bash
ls -la /home/ext_hajekim_google_com/.gemini/skills/ | grep thinking
head -5 /home/ext_hajekim_google_com/.gemini/skills/thinking-skills/SKILL.md
```

- [ ] **Step 3: Commit**

```bash
cd /home/ext_hajekim_google_com/sandbox/vertexai-skills
git add .
git commit -m "feat: deploy thinking-skills via symlink to ~/.gemini/skills"
```

---

# PART B: grounding-skills

---

## Task B-1: grounding-skills — frontmatter + 환경 설정

**Files:**
- Create: `skills/grounding-skills/SKILL.md`

- [ ] **Step 1: 디렉토리 생성 및 SKILL.md 작성**

```bash
mkdir -p /home/ext_hajekim_google_com/sandbox/vertexai-skills/skills/grounding-skills
```

`skills/grounding-skills/SKILL.md`를 아래 내용으로 생성:

````markdown
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
````

- [ ] **Step 2: 파일 확인**

```bash
head -20 /home/ext_hajekim_google_com/sandbox/vertexai-skills/skills/grounding-skills/SKILL.md
```

- [ ] **Step 3: Commit**

```bash
cd /home/ext_hajekim_google_com/sandbox/vertexai-skills
git add skills/grounding-skills/SKILL.md
git commit -m "feat: scaffold grounding-skills SKILL.md with env setup"
```

---

## Task B-2: grounding-skills — § 1 Google Search 그라운딩

- [ ] **Step 1: § 1 섹션 추가**

````markdown

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
````

- [ ] **Step 2: Commit**

```bash
cd /home/ext_hajekim_google_com/sandbox/vertexai-skills
git add skills/grounding-skills/SKILL.md
git commit -m "feat: add §1 Google Search grounding to grounding-skills"
```

---

## Task B-3: grounding-skills — § 2 Google Maps 그라운딩

- [ ] **Step 1: § 2 섹션 추가**

````markdown

---

## § 2. Google Maps 그라운딩

### 언제: 장소, 지역 정보, 주변 검색이 필요할 때

```python
from google import genai
from google.genai import types
from google.genai.types import (
    GenerateContentConfig,
    GoogleMaps,
    HttpOptions,
    Tool,
)

client = genai.Client(http_options=HttpOptions(api_version="v1"))

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="Where can I get the best espresso near me?",
    config=GenerateContentConfig(
        tools=[
            Tool(google_maps=GoogleMaps(
                enable_widget=False,   # True: 지도 위젯 토큰 반환
            ))
        ],
        tool_config=types.ToolConfig(
            retrieval_config=types.RetrievalConfig(
                lat_lng=types.LatLng(
                    latitude=37.5665,    # 서울 위도
                    longitude=126.9780,  # 서울 경도
                ),
                language_code="ko_KR",
            ),
        ),
    ),
)

print(response.text)
```

### GoogleMaps 파라미터

| 파라미터 | 타입 | 설명 |
|---------|------|------|
| `enable_widget` | `bool` | `True`이면 응답에 지도 위젯 토큰 포함 |
| `lat_lng.latitude` | `float` | 검색 기준 위도 |
| `lat_lng.longitude` | `float` | 검색 기준 경도 |
| `language_code` | `str` | 응답 언어 코드 (예: `"ko_KR"`, `"en_US"`) |

> **원칙:**
> - Maps 출처는 생성된 콘텐츠 바로 다음에 반드시 표시해야 한다.
> - Gemini 3 Pro / Gemini 3 Pro Image는 일일 5,000 쿼리 제한.
> - 금지 지역: 중국, 쿠바, 이란, 북한, 시리아 등.
````

- [ ] **Step 2: Commit**

```bash
cd /home/ext_hajekim_google_com/sandbox/vertexai-skills
git add skills/grounding-skills/SKILL.md
git commit -m "feat: add §2 Google Maps grounding to grounding-skills"
```

---

## Task B-4: grounding-skills — § 3 Vertex AI Search 그라운딩

- [ ] **Step 1: § 3 섹션 추가**

````markdown

---

## § 3. Vertex AI Search 그라운딩

### 언제: 내 문서나 웹사이트 데이터 스토어로 RAG를 구성할 때

```python
from google import genai
from google.genai.types import (
    GenerateContentConfig,
    HttpOptions,
    Retrieval,
    Tool,
    VertexAISearch,
)

client = genai.Client(http_options=HttpOptions(api_version="v1"))

# 데이터 스토어 경로 형식:
# projects/{PROJECT_ID}/locations/global/collections/default_collection/dataStores/{DATASTORE_ID}
DATASTORE_PATH = "projects/my-project/locations/global/collections/default_collection/dataStores/my-store"

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="What are the key features described in the documentation?",
    config=GenerateContentConfig(
        tools=[
            Tool(
                retrieval=Retrieval(
                    vertex_ai_search=VertexAISearch(
                        datastore=DATASTORE_PATH,
                    )
                )
            )
        ],
    ),
)

print(response.text)
# 출처 확인
for chunk in response.candidates[0].grounding_metadata.grounding_chunks:
    print(chunk.retrieved_context.uri)
```

### 사전 요구사항

```bash
# 1. AI Applications API 활성화
gcloud services enable discoveryengine.googleapis.com

# 2. IAM 권한 부여
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="user:USER_EMAIL" \
  --role="roles/discoveryengine.viewer"
```

### 파라미터

| 파라미터 | 타입 | 설명 |
|---------|------|------|
| `VertexAISearch.datastore` | `str` | 데이터 스토어 전체 리소스 경로 |

> **원칙:**
> - 최대 10개 데이터 소스를 동시에 사용할 수 있다.
> - Google Search 그라운딩과 병행 사용 가능.
> - Gemini 2.5 이상에서는 `confidence_scores`가 제공되지 않는다.
````

- [ ] **Step 2: Commit**

```bash
cd /home/ext_hajekim_google_com/sandbox/vertexai-skills
git add skills/grounding-skills/SKILL.md
git commit -m "feat: add §3 Vertex AI Search grounding to grounding-skills"
```

---

## Task B-5: grounding-skills — § 4 Elasticsearch 그라운딩

- [ ] **Step 1: § 4 섹션 추가**

````markdown

---

## § 4. Elasticsearch 그라운딩

### 언제: 기존 Elasticsearch 인덱스를 그라운딩 소스로 활용할 때

```python
from google import genai
from google.genai.types import (
    Elasticsearch,
    GenerateContentConfig,
    HttpOptions,
    Retrieval,
    Tool,
)

client = genai.Client(http_options=HttpOptions(api_version="v1"))

ES_ENDPOINT = "https://your-cluster.es.io:443"
ES_API_KEY = "your-elasticsearch-api-key"
INDEX_NAME = "your-index"
SEARCH_TEMPLATE = "your-search-template"

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="What are the main features of product X?",
    config=GenerateContentConfig(
        tools=[
            Tool(
                retrieval=Retrieval(
                    external_api=Elasticsearch(
                        api_spec="ELASTIC_SEARCH",
                        endpoint=ES_ENDPOINT,
                        api_auth={
                            "apiKeyConfig": {
                                "apiKeyString": f"ApiKey {ES_API_KEY}"   # "ApiKey " 접두사 필수
                            }
                        },
                        elastic_search_params={
                            "index": INDEX_NAME,
                            "searchTemplate": SEARCH_TEMPLATE,
                            "numHits": 5,            # 반환할 결과 수
                        },
                    )
                )
            )
        ],
    ),
)

print(response.text)
```

### Elasticsearch 파라미터

| 파라미터 | 설명 |
|---------|------|
| `api_spec` | 반드시 `"ELASTIC_SEARCH"` |
| `endpoint` | Elasticsearch 클러스터 URL |
| `apiKeyString` | 형식: `"ApiKey <your-key>"` — 접두사 `"ApiKey "` 필수 |
| `index` | 검색할 인덱스 이름 |
| `searchTemplate` | Elasticsearch 검색 템플릿 이름 |
| `numHits` | 반환할 결과 수 |

> **원칙:**
> - `apiKeyString` 값은 반드시 `"ApiKey "` 접두사를 포함해야 한다.
> - 텍스트 입력만 지원 (멀티모달 입력 불가).
> - 최대 10개 데이터 소스를 동시에 사용할 수 있다.
````

- [ ] **Step 2: Commit**

```bash
cd /home/ext_hajekim_google_com/sandbox/vertexai-skills
git add skills/grounding-skills/SKILL.md
git commit -m "feat: add §4 Elasticsearch grounding to grounding-skills"
```

---

## Task B-6: grounding-skills — § 5 Custom Search API 그라운딩

- [ ] **Step 1: § 5 섹션 추가**

````markdown

---

## § 5. Custom Search API 그라운딩

### 언제: 자체 검색 API를 그라운딩 소스로 연결할 때

```python
from google import genai
from google.genai.types import (
    ExternalApi,
    GenerateContentConfig,
    HttpOptions,
    Retrieval,
    Tool,
)

client = genai.Client(http_options=HttpOptions(api_version="v1"))

API_ENDPOINT = "https://your-api-gateway.example.com/v0/search"
API_KEY = "your-api-key"

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="What are the return policy details?",
    config=GenerateContentConfig(
        tools=[
            Tool(
                retrieval=Retrieval(
                    external_api=ExternalApi(
                        api_spec="SIMPLE_SEARCH",   # 현재 유일한 허용 값
                        endpoint=API_ENDPOINT,
                        api_auth={
                            "apiKeyConfig": {
                                "apiKeyString": API_KEY
                            }
                        },
                    )
                )
            )
        ],
    ),
)

print(response.text)
```

### 자체 API 계약 (필수 준수)

Gemini가 POST로 호출하는 요청 형식:
```json
{ "query": "검색어 문자열" }
```

응답으로 반환해야 하는 형식:
```json
[
  { "snippet": "관련 정보 텍스트", "uri": "출처 URL" },
  { "snippet": "두 번째 정보", "uri": "출처2 URL" }
]
```
결과가 없으면 빈 배열 `[]` 반환.

### ExternalApi 파라미터

| 파라미터 | 설명 |
|---------|------|
| `api_spec` | 반드시 `"SIMPLE_SEARCH"` |
| `endpoint` | API Gateway 엔드포인트 URL |
| `apiKeyConfig.apiKeyString` | Gemini가 `?key=` 쿼리 파라미터로 전달할 API 키 |

> **원칙:**
> - API 응답 지연이 길어지면 전체 Gemini 응답 시간이 증가한다.
> - `snippet`의 품질이 그라운딩 응답 품질을 직접 결정한다.
> - 인증은 API 키 방식만 지원된다.
````

- [ ] **Step 2: Commit**

```bash
cd /home/ext_hajekim_google_com/sandbox/vertexai-skills
git add skills/grounding-skills/SKILL.md
git commit -m "feat: add §5 custom search API grounding to grounding-skills"
```

---

## Task B-7: grounding-skills — § 6 Parallel Web Search + § 7 Enterprise Web Search

- [ ] **Step 1: § 6, § 7 섹션 추가**

````markdown

---

## § 6. Parallel Web Search 그라운딩

### 언제: Parallel AI의 LLM 최적화 웹 인덱스를 활용할 때

> **참고:** Parallel Web Search는 현재 REST API로만 사용할 수 있다.

```python
import json
import subprocess

PROJECT_ID = "your-project-id"
LOCATION = "us-central1"
MODEL_ID = "gemini-2.5-flash"
PARALLEL_API_KEY = "your-parallel-api-key"

request_body = {
    "contents": [{"role": "user", "parts": [{"text": "What is the latest news about AI?"}]}],
    "tools": [{
        "parallelAiSearch": {
            "api_key": PARALLEL_API_KEY,
            "customConfigs": {
                "source_policy": {
                    "exclude_domains": [],
                    "include_domains": [],
                },
                "excerpts": {
                    "max_chars_per_result": 30000,   # 1,000~100,000
                    "max_chars_total": 100000,        # 1,000~1,000,000
                },
                "max_results": 10,                   # 1~20
            }
        }
    }],
}

# REST API 호출 예시 (gcloud 인증 사용)
# curl -X POST \
#   -H "Authorization: Bearer $(gcloud auth print-access-token)" \
#   -H "Content-Type: application/json" \
#   -d '<request_body>' \
#   "https://LOCATION-aiplatform.googleapis.com/v1/projects/PROJECT_ID/locations/LOCATION/publishers/google/models/MODEL_ID:generateContent"
```

### Parallel Web Search 파라미터

| 파라미터 | 기본값 | 범위 | 설명 |
|---------|--------|------|------|
| `api_key` | — | — | Parallel AI API 키 (필수) |
| `exclude_domains` | — | 최대 10개 | 제외할 도메인 |
| `include_domains` | — | 최대 10개 | 포함할 도메인 (제한) |
| `max_chars_per_result` | 30,000 | 1,000~100,000 | 결과당 최대 문자 수 |
| `max_chars_total` | 100,000 | 1,000~1,000,000 | 전체 최대 문자 수 |
| `max_results` | 10 | 1~20 | 최대 결과 수 |

> **원칙:**
> - 분당 60 프롬프트 기본 할당량.
> - Parallel의 별도 이용약관이 적용된다.
> - Pre-GA 서비스로 SLA가 없다.

---

## § 7. Enterprise Web Search 그라운딩

### 언제: 의료·금융·공공 분야 등 컴플라이언스 규제 환경에서 웹 그라운딩이 필요할 때

```python
from google import genai
from google.genai.types import (
    EnterpriseWebSearch,
    GenerateContentConfig,
    HttpOptions,
    Tool,
)

client = genai.Client(http_options=HttpOptions(api_version="v1"))

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="When is the next total solar eclipse in the United States?",
    config=GenerateContentConfig(
        tools=[
            Tool(enterprise_web_search=EnterpriseWebSearch())   # 파라미터 없음
        ],
    ),
)

print(response.text)
```

### Google Search vs Enterprise Web Search 비교

| 항목 | Google Search | Enterprise Web Search |
|------|--------------|----------------------|
| 인덱스 범위 | 전체 웹 (더 넓음) | 규제 산업 최적화 |
| 최신성 | 실시간 | 6시간/24시간 업데이트 |
| 데이터 로깅 | 표준 | 없음 |
| VPC-SC 지원 | — | ✅ |
| CMEK | — | 해당 없음 |
| 일일 쿼리 제한 | — | 5,000 (Gemini 3 Pro/Pro Image) |

> **원칙:**
> - `EnterpriseWebSearch()`는 파라미터가 없다.
> - Search Suggestion(`webSearchQueries`)은 반드시 앱에 표시해야 한다.
> - 컴플라이언스가 필요 없다면 더 넓은 인덱스를 제공하는 Google Search를 사용한다.
````

- [ ] **Step 2: 전체 섹션 구조 확인**

```bash
grep "^## " /home/ext_hajekim_google_com/sandbox/vertexai-skills/skills/grounding-skills/SKILL.md
```

Expected:
```
## 환경 설정
## § 1. Google Search 그라운딩
## § 2. Google Maps 그라운딩
## § 3. Vertex AI Search 그라운딩
## § 4. Elasticsearch 그라운딩
## § 5. Custom Search API 그라운딩
## § 6. Parallel Web Search 그라운딩
## § 7. Enterprise Web Search 그라운딩
```

- [ ] **Step 3: Commit**

```bash
cd /home/ext_hajekim_google_com/sandbox/vertexai-skills
git add skills/grounding-skills/SKILL.md
git commit -m "feat: add §6 parallel web search and §7 enterprise web search to grounding-skills"
```

---

## Task B-8: grounding-skills — 심볼릭 링크 생성 및 최종 검증

- [ ] **Step 1: 심볼릭 링크 생성**

```bash
ln -s /home/ext_hajekim_google_com/sandbox/vertexai-skills/skills/grounding-skills \
      /home/ext_hajekim_google_com/.gemini/skills/grounding-skills
```

- [ ] **Step 2: 전체 스킬 최종 확인**

```bash
ls -la /home/ext_hajekim_google_com/.gemini/skills/ | grep -E "thinking|grounding"
echo "--- thinking-skills ---"
grep "^## " /home/ext_hajekim_google_com/.gemini/skills/thinking-skills/SKILL.md
echo "--- grounding-skills ---"
grep "^## " /home/ext_hajekim_google_com/.gemini/skills/grounding-skills/SKILL.md
```

- [ ] **Step 3: git log 확인**

```bash
cd /home/ext_hajekim_google_com/sandbox/vertexai-skills && git log --oneline | head -20
```

- [ ] **Step 4: 최종 Commit**

```bash
cd /home/ext_hajekim_google_com/sandbox/vertexai-skills
git add .
git commit -m "feat: deploy grounding-skills via symlink to ~/.gemini/skills"
```
