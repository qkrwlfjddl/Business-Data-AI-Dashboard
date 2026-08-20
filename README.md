# 통합 성과 분석 대시보드

온라인 교육 사업부의 매출·마케팅·콘텐츠 성과를 **한 곳에서** 조회·분석하고,  
AI가 자동으로 리포트를 생성·보관·비교하는 풀스택 데이터 플랫폼.

> 📹 시연 영상: (추후 링크 추가)

<!-- 스크린샷: 대시보드 메인 화면 (연간 매출 탭) -->
![대시보드 메인](docs/images/screenshot_main.png)

---

## 왜 만들었는가

| 기존 (Before) | 도입 후 (After) |
|--------------|----------------|
| 매출은 SAP, 마케팅은 GA4, 교재는 서점 사이트에서 각각 조회 | 하나의 대시보드에서 전부 확인 |
| 주간 보고서를 수동으로 작성 (2~3시간) | AI가 30초 만에 13개 섹션 자동 생성 |
| 과거 분석 결과가 개인 PC에 흩어져 있음 | BQ에 보관 → RAG 챗봇으로 언제든 검색 |
| "전년 대비 왜 떨어졌지?" 분석에 반나절 | AI가 증가/감소 요인을 교차 분석해서 즉시 제공 |
| 10명이 동시에 쓰면 느려지거나 튕김 | Cloud Run 자동 스케일링으로 동시 처리 |

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
