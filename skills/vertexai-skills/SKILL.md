---
name: vertexai-skills
description: Use when working with google-genai Python SDK on Vertex AI — code imports `from google import genai`, or requests involving "Vertex AI", "Gemini on Vertex", "function calling", "structured output", "code execution", "system instruction", "generation parameters". 한국어: "버텍스 AI", "함수 호출", "구조화된 출력", "코드 실행", "시스템 지시", "생성 파라미터", "제미나이 버텍스".
version: 1.0.0
---

# Vertex AI Skills — google-genai Python SDK 레퍼런스

## 환경 설정

### 필수 환경변수

```bash
export GOOGLE_CLOUD_PROJECT="your-project-id"
export GOOGLE_CLOUD_LOCATION="us-central1"   # "global"은 일부 모델만 지원
export GOOGLE_GENAI_USE_VERTEXAI="True"
```

### 클라이언트 초기화 (모듈 수준에서 한 번만)

```python
from google import genai
from google.genai.types import HttpOptions

client = genai.Client(http_options=HttpOptions(api_version="v1"))
```

### 대안: 직접 프로젝트 지정

```python
client = genai.Client(
    vertexai=True,
    project="your-project-id",
    location="us-central1",
)
```

> **원칙:** `client`는 모듈 최상단에서 한 번 초기화한다. 함수마다 재생성하지 않는다.
> 모든 설정은 `GenerateContentConfig`에 통합한다.

---

## § 1. 텍스트 생성

### 언제: 단순 텍스트 응답이 필요할 때

### 논스트리밍

```python
from google import genai
from google.genai.types import HttpOptions

client = genai.Client(http_options=HttpOptions(api_version="v1"))

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="How does AI work?",
)
print(response.text)
```

### 스트리밍 (멀티턴 채팅)

```python
chat = client.chats.create(model="gemini-2.5-flash")

for chunk in chat.send_message_stream("Why is the sky blue?"):
    print(chunk.text, end="")

# 이어서 대화 계속
for chunk in chat.send_message_stream("Tell me more."):
    print(chunk.text, end="")
```

> **선택 기준:**
> - 응답을 즉시 화면에 표시해야 하면 → 스트리밍
> - 응답 전체를 처리한 후 사용해야 하면 → 논스트리밍

---

## § 2. 시스템 지시 (System Instruction)

### 언제: 모델의 역할, 말투, 출력 형식을 고정해야 할 때

```python
from google import genai
from google.genai.types import HttpOptions, GenerateContentConfig

client = genai.Client(http_options=HttpOptions(api_version="v1"))

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="Write a haiku about the sea.",
    config=GenerateContentConfig(
        system_instruction="You are a poet. Always respond in Korean.",
    ),
)
print(response.text)
```

### 멀티턴 채팅에서의 시스템 지시

```python
chat = client.chats.create(
    model="gemini-2.5-flash",
    config=GenerateContentConfig(
        system_instruction="You are a helpful assistant that responds only in bullet points.",
    ),
)

response = chat.send_message("What are the benefits of exercise?")
print(response.text)
```

> **원칙:** 시스템 지시는 `GenerateContentConfig.system_instruction`에 설정한다.
> 프롬프트 앞에 직접 붙이지 않는다.

---

## § 3. 함수 호출 (Function Calling)

### 언제: 외부 API, 데이터베이스, 실시간 정보를 모델과 연결해야 할 때

### Step 1: 함수 선언

```python
from google import genai
from google.genai import types
from google.genai.types import HttpOptions, GenerateContentConfig

client = genai.Client(http_options=HttpOptions(api_version="v1"))

get_weather_func = types.FunctionDeclaration(
    name="get_current_weather",
    description="Get the current weather in a given location.",
    parameters={
        "type": "object",
        "properties": {
            "location": {
                "type": "string",
                "description": "City name, e.g. 'Seoul'",
            }
        },
        "required": ["location"],
    },
)
```

### Step 2: 모델에 도구 전달 및 함수 호출 응답 수신

```python
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="What is the weather in Seoul?",
    config=GenerateContentConfig(
        tools=[types.Tool(function_declarations=[get_weather_func])],
    ),
)

function_call = response.function_calls[0]
print(function_call.name)   # "get_current_weather"
print(function_call.args)   # {"location": "Seoul"}
```

### Step 3: 함수 실행 → 결과 반환

```python
# 실제 함수 실행 (예시)
api_result = {"location": "Seoul", "temperature": 15, "condition": "Cloudy"}

# 결과를 모델에 반환
final_response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[
        types.Content(role="user", parts=[types.Part(text="What is the weather in Seoul?")]),
        response.candidates[0].content,  # 모델의 함수 호출 요청
        types.Content(
            role="user",
            parts=[
                types.Part.from_function_response(
                    name=function_call.name,
                    response={"contents": api_result},
                )
            ],
        ),
    ],
    config=GenerateContentConfig(
        tools=[types.Tool(function_declarations=[get_weather_func])],
    ),
)
print(final_response.text)
```

> **원칙:**
> - `response.function_calls`로 함수 호출 여부를 확인한다.
> - 함수 결과는 반드시 `Part.from_function_response()`로 감싸서 반환한다.
> - 한 요청에 최대 512개 함수 선언 가능.

---

## § 4. 구조화된 출력 (Controlled Output)

### 언제: JSON 형식의 일관된 응답이 필요할 때 (파싱, 데이터 처리 등)

```python
from google import genai
from google.genai.types import HttpOptions, GenerateContentConfig

client = genai.Client(http_options=HttpOptions(api_version="v1"))

response_schema = {
    "type": "ARRAY",
    "items": {
        "type": "OBJECT",
        "properties": {
            "name": {"type": "STRING"},
            "price": {"type": "NUMBER"},
            "in_stock": {"type": "BOOLEAN"},
        },
        "required": ["name", "price", "in_stock"],
    },
}

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="List 3 popular fruits with their approximate price per kg and availability.",
    config=GenerateContentConfig(
        response_mime_type="application/json",
        response_schema=response_schema,
    ),
)

import json
data = json.loads(response.text)
print(data)
# [{"name": "Apple", "price": 3.5, "in_stock": true}, ...]
```

### 지원 타입

| JSON Schema 타입 | Python 예시 |
|-----------------|-------------|
| `STRING` | 문자열 |
| `NUMBER` | 정수/실수 |
| `BOOLEAN` | True/False |
| `ARRAY` | 리스트 |
| `OBJECT` | 딕셔너리 |

> **원칙:**
> - `response_mime_type`은 반드시 `"application/json"`으로 설정한다.
> - `required` 필드를 명시해야 모델이 해당 필드를 반드시 채운다.
> - 응답은 `json.loads(response.text)`로 파싱한다.
