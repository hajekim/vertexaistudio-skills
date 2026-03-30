# Design Spec: thinking-skills & grounding-skills

**Date:** 2026-03-30
**Status:** Approved

---

## Overview

두 개의 Gemini CLI 스킬 파일을 추가한다:

1. **`thinking-skills`** — Gemini 사고(추론) 모델 제어 + Thought Signatures
2. **`grounding-skills`** — 7가지 그라운딩 소스 연동 레퍼런스

기존 프로젝트 규칙을 그대로 따른다:
- `google-genai` Python SDK 전용 (`google-cloud-aiplatform` 제외)
- `genai.Client(http_options=HttpOptions(api_version="v1"))` 표준 초기화
- `~/.gemini/skills/`에 심볼릭 링크 배포

---

## Skill 1: thinking-skills

### Frontmatter

```yaml
name: thinking-skills
description: Use when working with thinking/reasoning models on Vertex AI using google-genai Python SDK — imports ThinkingConfig, or requests involving "thinking budget", "thinking level", "thought signatures", "reasoning model", "extended thinking". 한국어: "사고 모드", "추론 모델", "생각 예산", "사고 시그니처", "씽킹".
version: 1.0.0
```

### Sections (4개)

| 섹션 | 내용 |
|------|------|
| 환경 설정 | 클라이언트 초기화, 모델 목록, thinking_budget vs thinking_level 선택 기준 |
| § 1. Thinking Budget 설정 | Gemini 2.5용: thinking_budget 파라미터, 0으로 비활성화, -1(dynamic) |
| § 2. Thinking Level 설정 | Gemini 3+용: ThinkingLevel 열거형 (MINIMAL/LOW/MEDIUM/HIGH) |
| § 3. Thought 요약 보기 | include_thoughts=True, part.thought로 접근 |
| § 4. Thought Signatures | 함수 호출 시 자동 서명 처리, 멀티턴 대화 예제 |

### 모델 목록

```
gemini-2.5-flash     — thinking_budget 지원 (1~24,576 tokens)
gemini-2.5-pro       — thinking_budget 지원 (128~32,768 tokens), 비활성화 불가
gemini-2.5-flash-lite — thinking_budget 지원 (512~24,576 tokens)
gemini-3-flash       — thinking_level 지원
gemini-3.1-pro       — thinking_level 지원, 비활성화 불가
```

---

## Skill 2: grounding-skills

### Frontmatter

```yaml
name: grounding-skills
description: Use when working with grounding on Vertex AI using google-genai Python SDK — imports GoogleSearch/GoogleMaps/VertexAISearch/Elasticsearch/ExternalApi/EnterpriseWebSearch, or requests involving "grounding", "Google Search grounding", "RAG", "Vertex AI Search", "Elasticsearch grounding". 한국어: "그라운딩", "검색 기반 생성", "구글 검색 연동", "버텍스 검색", "환각 방지".
version: 1.0.0
```

### Sections (8개)

| 섹션 | Tool 클래스 | 핵심 파라미터 |
|------|------------|-------------|
| 환경 설정 | — | 그라운딩 방식 선택 가이드 |
| § 1. Google Search 그라운딩 | `Tool(google_search=GoogleSearch(...))` | `exclude_domains`, `dynamic_threshold` |
| § 2. Google Maps 그라운딩 | `Tool(google_maps=GoogleMaps(...))` | `enable_widget`, `lat_lng`, `language_code` |
| § 3. Vertex AI Search 그라운딩 | `Tool(retrieval=Retrieval(vertex_ai_search=...))` | `datastore` 경로 |
| § 4. Elasticsearch 그라운딩 | `Tool(retrieval=Retrieval(external_api=Elasticsearch(...)))` | `index`, `searchTemplate`, `numHits` |
| § 5. Custom Search API 그라운딩 | `Tool(retrieval=Retrieval(external_api=ExternalApi(...)))` | `api_spec="SIMPLE_SEARCH"`, `endpoint`, API 계약 |
| § 6. Parallel Web Search 그라운딩 | REST only (`parallelAiSearch`) | `max_results`, `exclude_domains` |
| § 7. Enterprise Web Search 그라운딩 | `Tool(enterprise_web_search=EnterpriseWebSearch())` | 파라미터 없음 |

### RAG 그라운딩 안내

`ground-responses-using-rag`는 `vertexai` SDK(`google-cloud-aiplatform`)를 사용하므로 이 스킬에 포함하지 않는다. § 3 Vertex AI Search 주석으로 안내한다.

---

## Deployment

```
skills/thinking-skills/SKILL.md    → ~/.gemini/skills/thinking-skills
skills/grounding-skills/SKILL.md   → ~/.gemini/skills/grounding-skills
```

---

## File Size 예상

| 스킬 | 예상 줄 수 |
|------|-----------|
| thinking-skills | ~280줄 |
| grounding-skills | ~420줄 |
