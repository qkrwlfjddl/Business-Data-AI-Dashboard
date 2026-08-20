<div align="center">

# Business Data & AI Dashboard

### 통합 성과 분석 · AI 리포트 · RAG 기반 업무용 데이터 플랫폼

<p>
  <img src="https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white">
  <img src="https://img.shields.io/badge/SQL-4479A1?style=flat-square&logoColor=white">
  <img src="https://img.shields.io/badge/Streamlit-FF4B4B?style=flat-square&logo=streamlit&logoColor=white">
  <img src="https://img.shields.io/badge/BigQuery-669DF6?style=flat-square&logo=googlebigquery&logoColor=white">
  <img src="https://img.shields.io/badge/Airflow-017CEE?style=flat-square&logo=apacheairflow&logoColor=white">
  <img src="https://img.shields.io/badge/Cloud%20Run-4285F4?style=flat-square&logo=googlecloud&logoColor=white">
  <img src="https://img.shields.io/badge/Gemini-8E75B2?style=flat-square&logo=googlegemini&logoColor=white">
  <img src="https://img.shields.io/badge/Vertex%20AI-4285F4?style=flat-square&logo=googlecloud&logoColor=white">
  <img src="https://img.shields.io/badge/RAG-412991?style=flat-square&logoColor=white">
</p>

<sub> 🔐 실제 업무 데이터와 소스 코드는 보안상 공개하지 않습니다.
공개 가능한 범위에서 시스템 구조, 데이터 흐름, 분석 로직 및 구현 내용을 정리했습니다.</sub>

</div>

---

온라인 교육 사업에 분산되어 있던 **매출 · 마케팅 · 콘텐츠 데이터를 BigQuery로 통합**하고,
통합 데이터를 기반으로 **성과 분석 → AI 리포트 → 과거 리포트 비교 → RAG 검색**까지 연결한 업무용 데이터 분석 플랫폼입니다.


----

## 구현 결과

| **🔗 데이터 통합** | **📊 통합 성과 분석** |
|---|---|
| **SAP · GA4 · 웹 · 콘텐츠 데이터**를 BigQuery로 통합 | 매출·GA·이벤트·교재·검색 트렌드를 **하나의 대시보드에서 분석** |

| **🤖 AI 분석 리포트** | **🔄 과거 리포트 비교** |
|---|---|
| 조회 데이터를 분석 컨텍스트로 구성하여 **AI 성과 분석 리포트 생성** | 현재 리포트와 저장된 과거 리포트를 **AI로 비교 분석** |

| **🧠 분석 지식 축적** | **🔍 RAG 검색** |
|---|---|
| AI 분석 결과 + 담당자 의견 + 분석 데이터 **구조화하여 저장** | Chunk + Embedding + `VECTOR_SEARCH` 기반 **자연어 검색** |

> **분산된 데이터를 먼저 통합하고 → 분석하고 → AI로 해석하고 → 결과를 다시 지식으로 축적하는 구조**

---

## 🎥 Demo

> 📹 실제 업무 시스템 시연 영상

[![Dashboard Demo](docs/images/screenshot_main.png)](YOUTUBE_URL)

---

## 🏗️ Architecture

```text
SAP / GA4 / Web / Content
          │
          ▼
    Airflow / DAGs
          │
          ▼
      BigQuery
          │
     ┌────┴────┐
     ▼         ▼
Dashboard   AI Analysis
               │
               ▼
           AI Report
          ┌────┴────┐
          ▼         ▼
      Comparison    RAG
                       │
                       ▼
                Natural Search
```

[전체 아키텍처 보기 →](docs/architecture.md)

---
## 📚 Documentation

[주요 기능과 화면 보기 →](docs/features.md)  
대시보드에서 제공하는 매출·이벤트·GA·AI 리포트·비교·RAG 기능을 정리했습니다.

[전체 아키텍처 보기 →](docs/architecture.md)  
데이터 통합부터 Dashboard, AI Analysis, Report Storage, Comparison, RAG까지 전체 시스템 흐름을 확인할 수 있습니다.

[AI / RAG 구현 보기 →](docs/ai-rag.md)  
선택적 AI 컨텍스트 구성, 리포트 저장, RAG Corpus, Vector Search, 과거 리포트 비교 구조를 정리했습니다.

