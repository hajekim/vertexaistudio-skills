# Image & Video Skills — Design Spec
**Date:** 2026-03-30
**Status:** Approved

---

## 1. 목적

Gemini CLI가 Vertex AI에서 이미지/비디오 생성 코드를 작성할 때 자동 로드되는 레퍼런스 가이드 스킬 2종. `google-genai` Python SDK 기준으로 Imagen 4 / Gemini Image / Veo 3.1 패턴을 제공한다.

---

## 2. 파일 구조

```
vertexai-skills/
├── skills/
│   ├── vertexai-skills/SKILL.md      ← 기존 유지
│   ├── image-skills/
│   │   └── SKILL.md                  ← 신규
│   └── video-skills/
│       └── SKILL.md                  ← 신규
└── docs/superpowers/specs/
    └── 2026-03-30-image-video-skills-design.md

# 배포: 심볼릭 링크
~/.gemini/skills/image-skills/ → skills/image-skills/
~/.gemini/skills/video-skills/ → skills/video-skills/
```

---

## 3. image-skills/SKILL.md

### 스킬 메타데이터

```yaml
name: image-skills
description: Use when working with image generation or editing on Vertex AI using
  google-genai Python SDK — imports Imagen or Gemini image models, or requests
  involving "image generation", "generate image", "edit image", "Imagen", "Gemini
  image". 한국어: "이미지 생성", "이미지 편집", "이미지 만들어줘", "아이마젠",
  "제미나이 이미지".
version: 1.0.0
```

### 내용 구조

| 섹션 | 내용 |
|------|------|
| 환경 설정 | 설치, 환경변수, 모델 선택 가이드 (Imagen 4 vs Gemini Image) |
| § 1. Imagen 4 이미지 생성 | `generate_images()` + `GenerateImagesConfig`, 결과 저장 |
| § 2. Gemini 이미지 생성 | `generate_content()` + `Modality.IMAGE`, 멀티모달 응답 처리 |
| § 3. Gemini 이미지 편집 | 이미지 + 프롬프트 입력, 멀티턴 편집 |
| § 4. Best Practices | 구체적 프롬프트, 반복 개선, 카메라 관점, 긍정적 표현 |
| § 5. Limitations | 언어 지원 목록, 입력 제한, 출력 한계, 알려진 동작 문제 |
| § 6. Responsible AI & Safety | 금지 콘텐츠 카테고리, Safety 필터 에러코드 처리 |

### 주요 모델

| 모델 | 용도 |
|------|------|
| `imagen-4.0-generate-001` | 최고 품질 정적 이미지 생성 |
| `gemini-2.5-flash-image` | 이미지 생성 + 편집, 경량 |
| `gemini-3-pro-image-preview` | 이미지 생성 + 편집, 최고 품질 |
| `gemini-3.1-flash-image-preview` | 이미지 생성 + 편집, 최신 |

---

## 4. video-skills/SKILL.md

### 스킬 메타데이터

```yaml
name: video-skills
description: Use when working with video generation on Vertex AI using google-genai
  Python SDK — imports Veo models, or requests involving "video generation",
  "generate video", "text to video", "image to video", "extend video", "Veo".
  한국어: "비디오 생성", "영상 만들기", "텍스트로 영상", "이미지로 영상", "비오",
  "영상 연장".
version: 1.0.0
```

### 내용 구조

| 섹션 | 내용 |
|------|------|
| 환경 설정 | 설치, 환경변수, GCS 버킷 필수 안내, 모델 목록 |
| § 1. 텍스트 → 비디오 | `generate_videos()` + 폴링 루프 패턴 |
| § 2. 이미지 → 비디오 | `Image(gcs_uri=...)` 입력 패턴 |
| § 3. 첫/마지막 프레임으로 비디오 | `image` + `last_frame` 파라미터 |
| § 4. 비디오 연장 (Extend) | `Video(uri=..., mime_type="video/mp4")` |
| § 5. 고급 기법 | 레퍼런스 이미지 가이드, 객체 삽입/제거 |
| § 6. 프롬프트 가이드 & Best Practices | 작성 원칙, 프롬프트 리라이터 끄기, 파라미터 가이드 |
| § 7. Responsible AI & Safety | 금지 콘텐츠, Safety 코드 |

### 주요 모델

| 모델 | 용도 |
|------|------|
| `veo-3.1-generate-001` | 최신 안정 버전 (권장) |
| `veo-3.1-fast-generate-001` | 빠른 생성 |
| `veo-3.0-generate-001` | 이전 안정 버전 |
| `veo-2.0-generate-001` | 레거시 (기존 고객) |

---

## 5. 설계 결정 사항

| 결정 | 이유 |
|------|------|
| image-skills / video-skills 분리 | 모델군 다름 (Imagen/Gemini vs Veo), 트리거 정밀도, 각 파일 크기 적정화 |
| Imagen 4와 Gemini Image 별도 섹션 | API가 완전히 다름 (`generate_images` vs `generate_content`) |
| Video는 폴링 루프 패턴 명시 | 비동기 LRO(Long Running Operation) — 모든 비디오 작업 공통 패턴 |
| Best Practices / Limitations / Responsible AI 포함 | 유저 요청 (전부 포함 B 선택) |
| Claude Code 심볼릭 링크 없음 | 이전 결정과 동일 (Gemini CLI 전용) |

---

## 6. 구현 범위

- [ ] `skills/image-skills/SKILL.md` 작성 (환경설정 + §1~§6)
- [ ] `skills/video-skills/SKILL.md` 작성 (환경설정 + §1~§7)
- [ ] `~/.gemini/skills/image-skills` 심볼릭 링크
- [ ] `~/.gemini/skills/video-skills` 심볼릭 링크
