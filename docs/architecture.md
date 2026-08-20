# Architecture

## 1. System Overview

이 시스템은 여러 업무 시스템에 분산되어 있던 데이터를 BigQuery로 통합한 뒤,
통합 데이터를 기반으로 성과 분석, AI 리포트 생성, 과거 리포트 비교,
RAG 기반 검색까지 연결한 업무용 데이터 분석 플랫폼입니다.

전체 구조는 다음과 같습니다.

```text
┌───────────────────────────────┐
│        Source Systems        │
│                               │
│ SAP / GA4 / Web / Content     │
└───────────────┬───────────────┘
                │
                │ Data Pipeline
                ▼
┌───────────────────────────────┐
│      HANA → BigQuery ETL      │
│                               │
│       Airflow / DAGs          │
│        Cloud Run Jobs         │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│           BigQuery            │
│                               │
│   Integrated Analytics Data   │
└───────────────┬───────────────┘
                │
        ┌───────┴────────┐
        ▼                ▼
┌───────────────┐ ┌───────────────┐
│   Dashboard   │ │  AI Analysis  │
│   Streamlit   │ │    Gemini     │
└───────────────┘ └───────┬───────┘
                          │
                          ▼
                  ┌────────────────┐
                  │   AI Report    │
                  └───────┬────────┘
                          │
                 ┌────────┴────────┐
                 ▼                 ▼
          ┌────────────┐     ┌────────────┐
          │ Comparison │     │    RAG     │
          │ Current ↔  │     │ Chunk +    │
          │ Past       │     │ Embedding  │
          └────────────┘     └──────┬─────┘
                                    │
                                    ▼
                             Natural Language
                                Search
```

---

## 2. Data Pipeline

### HANA → BigQuery

통합 분석 환경을 만들기 위해 원천 데이터를 BigQuery로 적재하는 데이터 파이프라인을 구성했습니다.

```text
Source Data
    ↓
Airflow / DAG
    ↓
Cloud Run Job
    ↓
BigQuery
```

데이터 적재 파이프라인은 별도의 프로젝트로 관리합니다.

👉 [HANA-BQ-ApacheAirflow](https://github.com/qkrwlfjddl/HANA-BQ-ApacheAirflow)

이 파이프라인은 본 대시보드가 사용하는 **통합 데이터 계층(Data Integration Layer)** 역할을 합니다.

---

## 3. Integrated Analytics Layer

BigQuery에는 대시보드와 AI 분석에서 사용할 수 있도록
여러 데이터 소스를 분석 목적에 맞게 집계한 데이터가 저장됩니다.

주요 분석 영역:

- 매출
- 상품
- 이벤트
- GA 성과
- 교재
- 검색 트렌드
- 환불 / 환급
- 상품 매핑

이렇게 데이터를 하나의 환경으로 통합함으로써
서로 다른 시스템의 데이터를 교차 분석할 수 있도록 구성했습니다.

---

## 4. Dashboard Layer

### Streamlit

통합된 BigQuery 데이터를 기반으로
업무 담당자가 매출과 마케팅 성과를 조회·분석할 수 있는 대시보드를 구현했습니다.

주요 기능:

- 연간 / 주간 / 일간 매출 분석
- KPI 달성률
- 전년 대비 증감률
- 상품별 Drill-down
- 이벤트 전환 분석
- GA 마케팅 성과
- 교재 순위
- 검색 트렌드

---

## 5. AI Analysis Layer

대시보드에서 조회된 데이터를 분석 목적에 따라 선택적으로 구성하여
AI 분석에 필요한 컨텍스트를 생성합니다.

```text
Dashboard Data
      ↓
Selected Analysis Areas
      ↓
Data Summary
      ↓
AI Prompt
      ↓
Gemini
      ↓
Structured AI Report
```

모든 데이터를 무조건 AI에게 전달하는 것이 아니라
사용자가 선택한 분석 영역에 따라 필요한 데이터만 포함하도록 구성했습니다.

분석 영역에는 다음과 같은 항목이 포함될 수 있습니다.

- KPI / 매출
- 상품
- GA
- 이벤트
- 교재
- 검색 트렌드
- 환불 / 환급
- 상품 매핑

---

## 6. AI Report Storage

생성된 AI 분석 리포트는 단순 텍스트로 끝내지 않고
향후 비교 분석과 검색에 사용할 수 있도록 BigQuery에 저장합니다.

저장 구조는 3개의 계층으로 구성됩니다.

| Layer | Purpose |
|---|---|
| `AI_REPORTS` | AI 리포트 섹션 및 담당자 의견 |
| `AI_REPORT_CHUNKS` | RAG 검색을 위한 Chunk + Embedding |
| `AI_REPORT_DATA` | 분석 당시 사용한 원본 데이터 Snapshot |

---

## 7. Report Lifecycle

AI 분석 리포트는 다음과 같은 흐름으로 관리됩니다.

```text
1. Dashboard Data 조회
        ↓
2. AI Report 생성
        ↓
3. 담당자 검토 및 의견 입력
        ↓
4. 리포트 저장
        ↓
5. RAG Corpus 생성
        ↓
6. 과거 리포트와 비교
        ↓
7. 이후 자연어 검색 및 재활용
```

즉, 분석 결과가 일회성 출력으로 끝나는 것이 아니라
**분석 → 검토 → 저장 → 비교 → 검색 → 재활용**으로 이어지는 구조입니다.

---

## 8. Report Comparison

현재 리포트와 과거에 저장된 리포트를 함께 AI에 전달하여
분석 결과가 시간에 따라 어떻게 변화했는지 비교합니다.

비교 항목:

- 개선된 항목
- 악화된 항목
- 반복되는 패턴
- 현재 확인해야 할 사항
- 과거 담당자의 의견

```text
Current Report
      +
Past Reports
      +
Previous Opinions
      ↓
   Gemini
      ↓
Comparison Report
```

---

## 9. RAG Layer

저장된 AI 리포트는 검색 가능한 단위로 분리하여
RAG용 Chunk와 Embedding으로 저장합니다.

```text
AI Report
    ↓
Section / Data Chunk
    ↓
Embedding
    ↓
BigQuery
```

사용자가 자연어로 질문하면:

```text
User Question
      ↓
Question Embedding
      ↓
BigQuery VECTOR_SEARCH
      ↓
Relevant Chunks
      ↓
Gemini
      ↓
Grounded Answer
```

별도의 Vector Database를 구축하지 않고
BigQuery의 `VECTOR_SEARCH`를 활용해 검색하도록 구성했습니다.

---

## 10. Related Project

### HANA → BigQuery Data Pipeline

본 프로젝트의 통합 데이터 계층을 담당하는 별도 프로젝트입니다.

👉 [HANA-BQ-ApacheAirflow](https://github.com/qkrwlfjddl/HANA-BQ-ApacheAirflow)

### Relationship

```text
HANA-BQ-ApacheAirflow
        │
        │ Data Integration
        ▼
Business-Data-AI-Dashboard
        │
        ├── Analytics
        ├── AI Report
        ├── Comparison
        └── RAG
```

즉,

**HANA-BQ-ApacheAirflow = 데이터를 모으는 계층**

**Business-Data-AI-Dashboard = 모인 데이터를 분석하고 AI로 활용하는 계층**

으로 역할을 구분합니다.
