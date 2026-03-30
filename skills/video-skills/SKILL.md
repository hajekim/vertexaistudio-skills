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
