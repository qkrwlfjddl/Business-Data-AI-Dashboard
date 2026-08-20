# Architecture

## 1. System Overview

여러 업무 시스템에 분산되어 있던 데이터를 BigQuery로 통합한 뒤,
통합 데이터를 기반으로 성과 분석, AI 리포트 생성, 과거 리포트 비교,
RAG 기반 검색까지 연결한 업무용 데이터 분석 플랫폼입니다.

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

## 2. Data Integration Layer

👉 [HANA-BQ-ApacheAirflow](https://github.com/qkrwlfjddl/HANA-BQ-ApacheAirflow)

이 파이프라인은 본 대시보드의 **Data Integration Layer** 역할을 합니다.

```text
Source Data
    ↓
Airflow / DAG
    ↓
Cloud Run Job
    ↓
BigQuery
```

> HANA-BQ-ApacheAirflow = 데이터를 통합하는 계층
>
> Business-Data-AI-Dashboard = 통합된 데이터를 분석하고 AI로 활용하는 계층

## 3. Integrated Analytics Layer

BigQuery에는 대시보드와 AI 분석에서 사용할 수 있도록 여러 데이터 소스를 분석 목적에 맞게 집계한 데이터가 저장됩니다.

주요 분석 영역:

- 매출
- 상품
- 이벤트
- GA 성과
- 교재
- 검색 트렌드
- 환불 / 환급
- 상품 매핑

## 4. Dashboard Layer

Streamlit 기반 대시보드에서 통합된 BigQuery 데이터를 조회합니다.

주요 분석 기능:

- 연간 / 주간 / 일간 매출 분석
- KPI 달성률
- 전년 대비 증감률
- 상품별 Drill-down
- 이벤트 전환 분석
- GA 마케팅 성과
- 교재 순위
- 검색 트렌드

## 5. AI Analysis Layer

대시보드에서 조회된 데이터 중 선택된 분석 영역을 중심으로 AI 컨텍스트를 구성합니다.

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

선택된 분석 영역에 해당하는 데이터만 프롬프트에 포함하도록 구성했습니다.

## 6. Report Storage Layer

| Layer | Purpose |
|---|---|
| `AI_REPORTS` | 리포트 섹션, AI 내용, 담당자 의견, 저장 메타데이터 |
| `AI_REPORT_CHUNKS` | RAG 검색용 Chunk + Embedding |
| `AI_REPORT_DATA` | 분석 당시 사용한 원본 데이터 Snapshot |

## 7. Report Lifecycle

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
7. 자연어 검색 / 재활용
```

**분석 → 검토 → 저장 → 비교 → 검색 → 재활용**으로 이어지는 구조입니다.

## 8. Report Comparison

현재 리포트와 과거에 저장된 리포트를 함께 AI에 전달하여 시간에 따른 변화와 반복 패턴을 분석합니다.

비교 항목:

- 개선된 항목
- 악화된 항목
- 반복되는 패턴
- 지금 확인해야 할 사항
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

## 9. RAG Layer

저장된 리포트와 분석 근거를 검색 가능한 단위로 구성합니다.

```text
AI Report / Data Summary
          ↓
      Text Chunk
          ↓
      Embedding
          ↓
   AI_REPORT_CHUNKS
          ↓
 BigQuery VECTOR_SEARCH
          ↓
   Relevant Chunks
          ↓
       Gemini
          ↓
   Grounded Answer
```

별도의 Vector Database 대신 BigQuery의 `VECTOR_SEARCH`를 활용합니다.
