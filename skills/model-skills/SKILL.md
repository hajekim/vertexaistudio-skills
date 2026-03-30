---
name: model-skills
description: >
  Vertex AI Studio 모델 선택 및 마이그레이션 가이드. 아래 키워드로 트리거된다:
  "model selection", "모델 선택", "model lifecycle", "모델 라이프사이클",
  "deprecated model", "deprecated 모델", "model migration", "모델 마이그레이션",
  "gemini migration", "1.5 to 2.5", "model retirement", "모델 은퇴",
  "which model", "어떤 모델", "모델 추천", "SDK migration", "SDK 마이그레이션"
version: 1.0.0
---

# Model Skills — Vertex AI Studio 모델 선택 및 마이그레이션 가이드

> 참고 문서: [Model versions and lifecycle](https://cloud.google.com/vertex-ai/generative-ai/docs/learn/model-versions) · [Migration guide](https://cloud.google.com/vertex-ai/generative-ai/docs/migrate)

---

## 모델 라이프사이클 개념

| 상태 | 의미 | 프로덕션 적합 여부 |
|------|------|-----------------|
| **Stable (GA)** | 공개 출시. 은퇴일 1개월 전부터 신규 접근 차단 | ✅ 권장 |
| **Preview** | 기능 검증 중. API 변경 가능성 있음 | ⚠️ 비권장 |
| **Deprecated** | 은퇴일 이후 404 오류 반환. 사용 불가 | ❌ 불가 |

---

## § 1. LLM 모델 선택

### 현재 Stable 모델 (2025년 기준)

| 모델 ID | 출시일 | 은퇴일 | 특징 |
|---------|--------|--------|------|
| `gemini-2.5-flash` | 2025-06-17 | 2026-06-17 | 속도·비용 균형 **(신규 프로젝트 기본 권장)** |
| `gemini-2.5-pro` | 2025-06-17 | 2026-06-17 | 고품질 복잡한 추론 |
| `gemini-2.5-flash-lite` | 2025-07-22 | 2026-07-22 | 경량, 비용 최적화 |
| `gemini-2.0-flash-001` | 2025-02-05 | 2026-06-01 | 기존 프로젝트만 — 신규 사용 불가 (2026-03-06 이후) |

### Retired 모델 — 이미 서비스 종료, 즉시 교체 필요

> API 호출 시 **404 오류** 반환. 사용 중이라면 지금 바로 교체해야 한다.

| 모델 ID | 은퇴일 | 대체 모델 |
|---------|--------|----------|
| `gemini-1.5-pro-001` | 2025-05-24 | `gemini-2.5-pro` |
| `gemini-1.5-pro-002` | 2025-09-24 | `gemini-2.5-pro` |
| `gemini-1.5-flash-001` | 2025-05-24 | `gemini-2.5-flash` |
| `gemini-1.5-flash-002` | 2025-09-24 | `gemini-2.5-flash` |
| `gemini-1.0-pro-001` | 2025-04-21 | `gemini-2.5-flash` |
| `gemini-1.0-pro-002` | 2025-04-21 | `gemini-2.5-flash` |
| `gemini-1.0-pro-vision-001` | 2025-04-21 | `gemini-2.5-flash` |

### 용도별 선택 기준

```
비용 최소화          → gemini-2.5-flash-lite
속도·품질 균형 (기본) → gemini-2.5-flash      ← 대부분의 경우 이것
고품질 추론          → gemini-2.5-pro
사고 모델 (예산 제어) → gemini-2.5-flash / gemini-2.5-pro  (thinking_budget)
사고 모델 (레벨 제어) → gemini-3-flash / gemini-3.1-pro    (thinking_level)
```

### 자동 업데이트 별칭(Alias) 주의

| 별칭 | 현재 가리키는 버전 | 주의사항 |
|------|-----------------|---------|
| `gemini-2.5-pro` | `gemini-2.5-pro-001` | 새 버전 출시 시 자동 업그레이드됨 |
| `gemini-2.5-flash` | `gemini-2.5-flash-001` | 동일 |
| `gemini-2.0-flash` | `gemini-2.0-flash-001` | 동일 |

> **프로덕션에서는 별칭 대신 `gemini-2.5-flash-001` 같은 버전 고정 ID 사용 권장.** 별칭은 자동 업그레이드되어 동작이 바뀔 수 있다.

---

## § 2. 이미지 모델 선택

### 현재 Stable 모델

| 모델 ID | API | 출시일 | 은퇴일 | 특징 |
|---------|-----|--------|--------|------|
| `imagen-4.0-generate-001` | `generate_images()` | 2025-08-14 | 2026-06-30 | 최고 품질 정적 이미지 **(권장)** |
| `imagen-3.0-generate-002` | `generate_images()` | 2025-01-29 | 2026-06-30 | Imagen 3 안정 버전 |
| `gemini-2.5-flash-image` | `generate_content()` | — | — | 이미지 생성+편집, 경량 |

### Preview 모델 (프로덕션 비권장)

| 모델 ID | API | 특징 |
|---------|-----|------|
| `gemini-3-pro-image-preview` | `generate_content()` | 최고 품질 생성+편집 |
| `gemini-3.1-flash-image-preview` | `generate_content()` | 최신, 빠른 생성+편집 |

### 용도별 선택 기준

```
최고 품질 정적 이미지    → imagen-4.0-generate-001   (generate_images)
이미지 생성 + 텍스트 혼합 → gemini-2.5-flash-image    (generate_content)
최신 편집 기능 (실험적)  → gemini-3.1-flash-image-preview
```

---

## § 3. 비디오 모델 선택

### 현재 Stable 모델

| 모델 ID | 출시일 | 은퇴일 | 특징 |
|---------|--------|--------|------|
| `veo-3.1-generate-001` | 2025-11-17 | 미정 | 최신 안정 버전 **(권장)** |
| `veo-3.1-fast-generate-001` | 2025-11-17 | 미정 | 빠른 생성, 낮은 비용 |
| `veo-3.0-generate-001` | 2025-07-29 | 2026-06-30 | 이전 안정 버전 |
| `veo-2.0-generate-001` | — | — | 객체 삽입/제거, `enhance_prompt` 전용 |

### 용도별 선택 기준

```
일반 비디오 생성         → veo-3.1-generate-001
빠른 생성 / 비용 절약    → veo-3.1-fast-generate-001
객체 삽입·제거           → veo-2.0-generate-001  (이 기능은 2.0 전용)
enhance_prompt 사용      → veo-2.0-generate-001
```

---

## § 4. 마이그레이션 가이드

### 4-1. SDK 마이그레이션 (필수)

Vertex AI SDK(`google-cloud-aiplatform`)는 2026년 6월 이후 Gemini를 지원하지 않는다. Gen AI SDK로 전환해야 한다.

```bash
# 기존 SDK 제거
pip uninstall google-cloud-aiplatform

# Gen AI SDK 설치
pip install --upgrade google-genai
```

```python
# Before (Vertex AI SDK)
import vertexai
from vertexai.generative_models import GenerativeModel

vertexai.init(project="my-project", location="us-central1")
model = GenerativeModel("gemini-1.5-flash")
response = model.generate_content("Hello")

# After (Gen AI SDK)
from google import genai
from google.genai.types import HttpOptions

client = genai.Client(http_options=HttpOptions(api_version="v1"))
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="Hello",
)
```

### 4-2. 모델 ID 변경

```python
# 1.5 → 2.5 직접 교체 (1.5는 이미 모두 은퇴 — 404 반환 중)
"gemini-1.5-flash-001"  →  "gemini-2.5-flash"   # 2025-05-24 종료
"gemini-1.5-flash-002"  →  "gemini-2.5-flash"   # 2025-09-24 종료
"gemini-1.5-pro-001"    →  "gemini-2.5-pro"     # 2025-05-24 종료
"gemini-1.5-pro-002"    →  "gemini-2.5-pro"     # 2025-09-24 종료

# 1.0 → 2.5 (1.0은 2025-04-21 이미 종료)
"gemini-1.0-pro-001"         →  "gemini-2.5-flash"
"gemini-1.0-pro-vision-001"  →  "gemini-2.5-flash"

# 2.0 → 2.5 (신규 프로젝트 — 2026-03-06부터 신규 접근 차단)
"gemini-2.0-flash-001"  →  "gemini-2.5-flash"
```

### 4-3. Breaking Changes 체크리스트

#### thinking 파라미터 (Gemini 3 이상으로 업그레이드 시)

```python
# Before (Gemini 2.5 — thinking_budget)
from google.genai.types import GenerateContentConfig, ThinkingConfig

config = GenerateContentConfig(
    thinking_config=ThinkingConfig(thinking_budget=1024),
)

# After (Gemini 3+ — thinking_level)
from google.genai.types import GenerateContentConfig, ThinkingConfig, ThinkingLevel

config = GenerateContentConfig(
    thinking_config=ThinkingConfig(thinking_level=ThinkingLevel.HIGH),
)
```

#### Top-K 파라미터 제거 (Gemini 1.0 Pro Vision 이후 미지원)

```python
# Before
config = GenerateContentConfig(top_k=40)  # 일부 모델에서 동작

# After — top_k 제거, temperature/top_p만 사용
config = GenerateContentConfig(
    temperature=1.0,
    top_p=0.95,
)
```

#### Dynamic Retrieval → Google Search Grounding 전환

```python
# Before (Dynamic Retrieval — deprecated)
from google.genai.types import DynamicRetrievalConfig, GoogleSearchRetrieval, Tool

tool = Tool(google_search_retrieval=GoogleSearchRetrieval(
    dynamic_retrieval_config=DynamicRetrievalConfig(dynamic_threshold=0.7),
))

# After (Google Search Grounding)
from google.genai.types import GoogleSearch, Tool

tool = Tool(google_search=GoogleSearch())
```

#### temperature 기본값 주의 (Gemini 3 이상)

```python
# Gemini 3+ 모델은 temperature 기본값 1.0 유지 권장
# 낮은 값으로 변경 시 반복 응답 또는 성능 저하 가능

config = GenerateContentConfig(
    temperature=1.0,  # 명시적으로 1.0 설정 (기본값 유지)
)
```

### 4-4. 권장 마이그레이션 절차 (7단계)

```
1. 평가 요구사항 문서화
   └─ 기존 평가 데이터 수집, RAG/Tool use/Agent 컴포넌트별 정리

2. 코드 업그레이드
   └─ Gen AI SDK 전환 → 모델 ID 변경 → Breaking changes 수정

3. 오프라인 평가
   └─ 기존 평가 반복 수행 (Gen AI Evaluation Service 활용)

4. 프롬프트 조정
   └─ Hill climbing으로 반복 개선, system instruction 점검 (모순 제거)

5. 로드 테스트
   └─ Provisioned Throughput 사전 구매 여부 확인

6. 온라인 평가 (선택)
   └─ A/B 테스트 또는 Canary 배포

7. 프로덕션 배포
```

### 4-5. 마이그레이션 전 확인사항

| 항목 | 내용 |
|------|------|
| **보안·컴플라이언스** | InfoSec/규제 승인 선행 |
| **지역 가용성** | 모델별 지원 리전 확인 (`us-central1` 권장) |
| **가격 변경** | Gemini 2+ 모델은 문자 기반 → 토큰 기반 청구 |
| **Fine-tuning** | 기존 튜닝 모델 재사용 불가, 새 튜닝 필요 |
| **이미지/PDF 토큰화** | Gemini 3+에서 Pan and Scan → 가변 시퀀스 길이로 변경 |
| **PDF 처리** | Gemini 3+에서 OCR 기본값 비활성화 |
| **Image Segmentation** | Gemini 3+에서 미지원 |

> **주의:** 철저한 테스트 없이 전체 마이그레이션을 한 번에 진행하지 말 것. 단계적 전환 권장.
