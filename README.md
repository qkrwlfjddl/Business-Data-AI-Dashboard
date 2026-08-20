
<div align="center">

# Business Data & AI Dashboard

온라인 교육 사업의 **매출 · 마케팅 · 콘텐츠 데이터를 BigQuery로 통합**하고,
통합 데이터를 기반으로 **성과 분석 → AI 리포트 → 과거 리포트 비교 → RAG 검색**까지 연결한 업무용 데이터 분석 플랫폼입니다.

<p>
  <img src="https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white">
  <img src="https://img.shields.io/badge/SQL-4479A1?style=flat-square&logo=databricks&logoColor=white">
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

## 🎯 Why

기존에는 SAP, GA4, 웹사이트 등 여러 시스템에 데이터가 분산되어 있어
각 데이터를 개별적으로 조회하고 교차 분석해야 했습니다.

또한 데이터가 분산되어 있어 AI가 사업 전체의 데이터를 함께 분석하기 어려웠습니다.

따라서 먼저 **분산된 데이터를 BigQuery로 통합**하고,
그 위에 대시보드·AI 분석·리포트 저장·비교·RAG를 연결했습니다.

### 핵심 흐름

```text
분산 데이터
    ↓
Airflow / DAG
    ↓
BigQuery 통합
    ↓
Dashboard
    ↓
AI Analysis
    ↓
AI Report
    ├── 과거 리포트 비교
    └── RAG 검색
```

---

## 🚀 Key Features

### 📊 Unified Performance Dashboard

매출 · 이벤트 · GA · 교재 · 검색 트렌드 등 여러 성과 데이터를 하나의 환경에서 조회하고 분석합니다.

### 🤖 AI Report

사용자가 선택한 분석 영역의 데이터를 AI 컨텍스트로 구성하여
성과 변화, 증가·감소 요인, Next Action을 포함한 분석 리포트를 생성합니다.

### 🔄 Report Comparison

현재 리포트와 과거에 저장된 리포트를 비교하여
개선·악화 항목, 반복 패턴, 주요 확인 사항을 분석합니다.

### 🔍 RAG Knowledge Base

AI 분석 결과, 담당자 의견, 분석에 사용된 데이터 요약을 저장하고
Chunk + Embedding 기반 검색을 통해 과거 분석 결과를 자연어로 다시 활용합니다.

---

## 🏗️ Architecture

```text
┌─────────────────────────────┐
│ SAP / GA4 / Web / Content   │
└──────────────┬──────────────┘
               │
         Airflow / DAG
               │
               ▼
      ┌─────────────────┐
      │    BigQuery     │
      │ Integrated Data │
      └────────┬────────┘
               │
       ┌───────┴────────┐
       ▼                ▼
  Dashboard         AI Analysis
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

---

## 👤 My Contribution

* 분산된 SAP / GA4 / 웹 데이터를 BigQuery 중심으로 통합
* Airflow / DAG 기반 데이터 적재 자동화
* Streamlit 기반 통합 성과 분석 대시보드 구현
* Gemini 기반 AI 분석 리포트 생성 구조 구현
* AI 리포트 + 담당자 의견 + 분석 데이터의 저장 구조 설계
* BigQuery `VECTOR_SEARCH` 기반 RAG 구현
* 현재 리포트와 과거 리포트의 AI 비교 분석 기능 구현

---

## 📸 Screenshots

![Dashboard Main](docs/images/screenshot_main.png)

| Annual Sales                                 | AI Report                                          |
| -------------------------------------------- | -------------------------------------------------- |
| ![Annual](docs/images/screenshot_annual.png) | ![AI Report](docs/images/screenshot_ai_report.png) |

---

## 📚 Documentation

* [Architecture](docs/architecture.md)
* [Features](docs/features.md)
* [AI / RAG](docs/ai-rag.md)
* [My Contribution](docs/contribution.md)

---
