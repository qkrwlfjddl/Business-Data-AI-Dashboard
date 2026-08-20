# Business Data & AI Dashboard

온라인 교육 사업의 **매출 · 마케팅 · 콘텐츠 데이터를 BigQuery로 통합**하고,
통합된 데이터를 기반으로 **성과 분석 → AI 리포트 생성 → 과거 리포트 비교 → RAG 검색**까지 연결한 업무용 데이터 분석 플랫폼입니다.

> 🔒 **Private Work Project**
> 실제 업무 데이터와 소스 코드는 보안상 공개하지 않습니다.
> 공개 가능한 범위에서 데이터 흐름, 시스템 구조, 분석 로직 및 구현 내용을 정리했습니다.

---

## Why

기존에는 매출·마케팅·콘텐츠 데이터가 여러 시스템에 분산되어 있어,
각 데이터를 개별적으로 조회하고 교차 분석해야 했습니다.

특히 데이터가 분산된 상태에서는 AI가 전체 사업 데이터를 한 번에 이해하고 분석하기 어려웠기 때문에,
먼저 **AI가 분석할 수 있는 통합 데이터 환경을 구축하는 것**을 목표로 했습니다.

| Before                             | After                       |
| ---------------------------------- | --------------------------- |
| SAP / GA4 / 웹사이트 등 데이터가 여러 시스템에 분산 | BigQuery 중심으로 통합            |
| 각 시스템을 개별 조회 후 사람이 교차 분석           | 하나의 대시보드에서 통합 분석            |
| 분산된 데이터로 AI 분석 범위가 제한됨             | 통합 데이터를 AI 분석 컨텍스트로 구성      |
| 분석 결과가 일회성 리포트로 끝남                 | 리포트 + 담당자 의견 + 분석 데이터 저장    |
| 과거 리포트를 직접 찾아 비교                   | AI 기반 현재 ↔ 과거 리포트 비교        |
| 과거 분석 지식이 파일에 분산                   | Chunk + Embedding 기반 RAG 검색 |

### 핵심

> **데이터를 먼저 통합하고, 그 위에 AI 분석을 구축한 뒤, 분석 결과를 다시 지식으로 축적하는 구조**

---

## Architecture

```text
[ SAP / GA4 / Web / Content ]
              │
              ▼
       Airflow / DAGs
              │
              ▼
        ┌───────────┐
        │ BigQuery  │
        │ 통합 저장소 │
        └─────┬─────┘
              │
       ┌──────┴──────┐
       ▼             ▼
 [ Dashboard ]   [ AI Analysis ]
       │             │
       │             ▼
       │        [ AI Report ]
       │             │
       │      ┌──────┴──────┐
       │      ▼             ▼
       │ [ Comparison ]   [ RAG ]
       │                   │
       └─────────┬─────────┘
                 ▼
          자연어 기반 검색
```

---

## Key Results

* 📊 여러 시스템에 분산된 데이터를 **하나의 분석 환경으로 통합**
* 🤖 통합 데이터를 기반으로 **AI 성과 리포트 자동 생성**
* 🔄 현재 리포트와 과거 리포트를 **AI 기반으로 비교 분석**
* 🧠 AI 분석 결과·담당자 의견·원본 분석 데이터를 **지식 형태로 축적**
* 🔍 축적된 분석 결과를 **RAG 기반 자연어 검색으로 재활용**

---

## 🎥 Demo

> 📹 시연 영상: 추후 추가 예정

![Dashboard Main](docs/images/screenshot_main.png)

---

## 주요 기능


# Business Data & AI Dashboard

온라인 교육 사업의 **매출 · 마케팅 · 콘텐츠 데이터를 하나의 분석 환경으로 통합**하고,
반복적인 성과 분석과 리포트 작성을 **AI 기반으로 자동화한 데이터 분석 플랫폼**입니다.

> **실무 프로젝트**
>
> 실제 업무에서 사용하는 데이터와 일부 구현 코드는 보안상 공개하지 않으며,
> 공개 가능한 범위에서 시스템 구조와 데이터 분석 방법을 정리했습니다.

---

## 🎯 Problem

기존 성과 분석 과정에서는 데이터 조회와 리포트 작성에 반복적인 수작업이 필요했습니다.

| Before                     | After                 |
| -------------------------- | --------------------- |
| SAP / GA4 / 콘텐츠 데이터를 각각 조회 | 하나의 대시보드에서 통합 조회      |
| 주간 보고서 수동 작성               | AI 기반 리포트 자동 생성       |
| 과거 분석 결과가 개인 PC에 분산        | BigQuery에 저장하여 검색·재활용 |
| 전년 대비 변화 원인 수작업 분석         | AI 기반 교차 분석           |
| 사용자 증가 시 서버 처리 부담          | Cloud Run 자동 스케일링     |

### 핵심 개선

**반복적인 데이터 조회 → 분석 → 리포트 작성 과정을 하나의 시스템으로 연결**

---

## 🚀 What I Built

### 📊 Unified Performance Dashboard

매출·이벤트·마케팅·콘텐츠 데이터를 통합하여 연간 / 주간 / 일간 성과를 조회하고 분석할 수 있도록 구현했습니다.

### 🤖 AI Report Generation

사용자가 선택한 분석 항목만 AI에 전달하여
성과 변화, 증가·감소 요인, 다음 액션을 자동으로 생성하도록 구성했습니다.

### 🔍 Report Search & RAG

생성된 리포트를 BigQuery에 구조화하여 저장하고,
Embedding + `VECTOR_SEARCH`를 활용해 과거 분석 결과를 자연어로 검색할 수 있도록 구현했습니다.

### 🔄 Report Comparison

현재 분석 결과와 과거 리포트를 비교하여
개선·악화 항목, 반복 패턴, Action Item을 자동으로 도출합니다.

---

## 📈 Key Results

| 항목        |                              개선 |
| --------- | ------------------------------: |
| 주간 리포트 작성 |               **2~3시간 → 약 30초** |
| 과거 리포트 검색 | 개인 PC 파일 검색 → **RAG 기반 자연어 검색** |
| 성과 분석     |         개별 시스템 조회 → **통합 대시보드** |
| 분석 비교     |           수작업 비교 → **AI 자동 비교** |

> ※ 수치는 실제 업무 환경에서의 측정값을 기준으로 작성했습니다.

---

## 🎥 Demo

> 📹 시연 영상: 추후 추가 예정

![Dashboard Main](docs/images/screenshot_main.png)

---

## 주요 기능

### 1. 매출 성과 분석 (연간 / 주간 / 일간)

<!-- 스크린샷: 연간 매출 테이블 + KPI 달성률 -->
![연간 매출](docs/images/screenshot_annual.png)

- 기간 설정(시작~끝)으로 원하는 구간만 조회
- KPI 달성률 자동 계산 (전년 × 배율)
- 전년 동기 대비 증감률
- LEVEL1~4 드릴다운 + 상품코드 필터

### 2. 이벤트 전환 분석

<!-- 스크린샷: 이벤트 탭 (무료→유료 전환 퍼널) -->
![이벤트](docs/images/screenshot_event.png)

- 무료 이벤트 → 회원가입 → 유료 구매 전환 퍼널
- 이벤트별 매출 기여도
- 연간/주간/일간 추이

### 3. GA 마케팅 성과

<!-- 스크린샷: GA 성과 탭 (소스/매체별 표) -->
![GA 성과](docs/images/screenshot_ga.png)

- 소스/매체별 세션·매출·전환율
- 네이버 SA 채널별 성과
- 캠페인 필터링

### 4. AI 리포트 자동 생성

<!-- 스크린샷: AI 리포트 생성 결과 (카드형 섹션) -->
![AI 리포트](docs/images/screenshot_ai_report.png)

- 체크한 분석 대상만 AI에게 전달 (토큰 절약)
- Gemini 3.5 Flash-Lite 기반 30~60초 내 생성
- 섹션별 카드형 UI + 담당자 의견 입력
- 미완료 월 자동 감지 ("진행 중이므로 KPI 미달로 분석하지 마세요")

### 5. 기존 리포트 불러오기 & 비교 분석

<!-- 스크린샷: 비교 분석 결과 카드 -->
![비교 분석](docs/images/screenshot_comparison.png)

- 보관된 리포트를 선택해 다시 조회
- 현재 리포트 vs 과거 리포트 AI 비교
- 개선/악화 항목, 반복 패턴, 액션 아이템 자동 도출

### 6. RAG 챗봇 (보관 데이터 검색·분석)

<!-- 스크린샷: 챗봇 질문 + 답변 + 근거 문서 -->
![RAG 챗봇](docs/images/screenshot_chatbot.png)

- 자연어로 질문 → 보관된 모든 리포트에서 벡터 검색
- 근거 기반 답변 (인용 번호 [1][2] 표시)
- 다른 담당자가 남긴 의견까지 활용

---

## 기술 아키텍처

### 데이터 흐름

| 단계 | 구성 요소 | 설명 |
|------|----------|------|
| 1. 원본 | SAP ZTIF9001, GA4 events | 매출 원장 + 웹 분석 데이터 |
| 2. ETL | dash_sales (Cloud Run Job) | SAP → SALES_DAILY 요약 테이블 |
| 2. ETL | dash_ga4 (Cloud Run Job) | GA4 → 일별 성과 요약 테이블 |
| 3. 저장소 | BigQuery (DASHBOARD) | 모든 집계·AI 결과·RAG 청크 저장 |
| 4. 대시보드 | Streamlit (Cloud Run Service) | 매출·이벤트·교재·GA·트렌드 시각화 |
| 5. AI 생성 | dash-ai-svc (Cloud Run Service) | Gemini 3.5 Flash-Lite 호출 |
| 6. 임베딩 | Vertex AI Embedding | text-multilingual-embedding-002 (768d) |

### 요청 흐름

```text
[사용자] → Streamlit → HTTP POST → dash-ai-svc → Gemini API → 응답 반환
                                                                    ↓
                                                         섹션별 파싱 → UI 렌더링
                                                                    ↓
                                              [보관 클릭] → BQ 3-레이어 저장 (RAG용)
```

### RAG 흐름

```text
[질문 입력] → 질문 임베딩 → BQ VECTOR_SEARCH → top-k 청크 → Gemini 종합 답변
```

---

## AI / RAG 구현 상세

### 선택적 섹션 생성

사용자가 체크한 분석 대상만 프롬프트에 포함:

| 체크 항목 | 생성 섹션 |
|-----------|----------|
| 매출 | 매출 추이, 매출 증가, 매출 감소 |
| 환불/환급 | 환불·환급 분석 |
| 상품별 매핑 | 상품 그룹 비교 |
| 교재 순위 | 교재 순위 분석 |
| GA 성과 | GA 채널 효율 분석 (SA 포함) |
| 검색 트렌드 | 검색 트렌드 분석 |
| 이벤트 | 이벤트 전환 분석 (무료→유료 퍼널) |
| 검색 트렌드 + 교재 순위 | 트렌드×교재 상관 분석 |
| (항상) | 종합 요약, Next Action 제안 |

### RAG 3-레이어 저장

| 레이어 | 테이블 | 용도 |
|--------|--------|------|
| 헤더 | AI_REPORTS | 섹션별 텍스트 + 담당자 의견 |
| 청크 | AI_REPORT_CHUNKS | 768d 임베딩 벡터 (검색용) |
| 스냅샷 | AI_REPORT_DATA | 원본 DataFrame JSON |

### 벡터 검색

```python
# BigQuery 네이티브 VECTOR_SEARCH — 별도 벡터DB 불필요
SELECT base.chunk_id, base.chunk_text, distance
FROM VECTOR_SEARCH(
  (SELECT * FROM AI_REPORT_CHUNKS WHERE bsark = @bsark),
  'embedding',
  (SELECT @query_embedding AS embedding),
  top_k => 8,
  distance_type => 'COSINE'
)
ORDER BY distance
```

### 비교 리포트

- 현재 리포트 vs 과거 보관 리포트 (최대 3건) 비교
- 5개 고정 섹션: 한눈에 보기, 개선/악화 항목, 반복 패턴, 액션 아이템
- 과거 담당자 의견도 AI에게 전달

---

## 기술 스택

| 영역 | 기술 |
|------|------|
| Frontend | Streamlit (Python) |
| Backend | Google Cloud Run (Service + Job) |
| Database | BigQuery (분석 + 벡터 검색 겸용) |
| AI/LLM | Google AI Studio — Gemini 3.5 Flash-Lite |
| Embedding | Vertex AI — text-multilingual-embedding-002 (768d) |
| Vector Search | BigQuery VECTOR_SEARCH (COSINE) |
| ETL | Python + BigQuery SQL (Cloud Run Job) |
| Auth | Google OAuth 2.0 + BQ 권한 테이블 |
| CI/CD | Cloud Build + gcloud CLI |

---

## 프로젝트 구조

```text
sales-dashboard/
├── app.py                    # Streamlit 메인 앱
├── prompts/                  # AI 프롬프트 템플릿
│   ├── report_prompt.py
│   └── comparison_prompt.py
├── dash_ai/                  # Cloud Run Service: AI 생성
│   ├── Dockerfile
│   └── server.py
├── dash_sales/               # Cloud Run Job: 매출 ETL
│   └── run_sales_etl.py
├── dash_ga4/                 # Cloud Run Job: GA4 ETL
│   └── run_ga_etl.py
├── sql/                      # BigQuery DDL
│   └── ai_rag_schema.sql
└── .streamlit/
```
