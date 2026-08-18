# 통합 성과 분석 대시보드 — 아키텍처 & AI/RAG 구현

온라인 교육 사업부의 매출·마케팅·콘텐츠 성과를 통합 분석하는 대시보드.  
AI 리포트 자동 생성, RAG 기반 챗봇, 비교 분석까지 포함한 풀스택 데이터 플랫폼.

---

## 프로젝트 구조

```
sales-dashboard/
├── app.py                    # Streamlit 메인 앱 (UI + 비즈니스 로직)
├── prompts/                  # AI 프롬프트 템플릿 (분리 관리)
│   ├── report_prompt.py      #   13개 섹션 리포트 생성
│   └── comparison_prompt.py  #   과거 리포트 비교 분석
├── dash_ai/                  # Cloud Run Job: AI 생성 워커
│   ├── Dockerfile
│   ├── requirements.txt
│   └── run_ai_job.py         #   BQ 큐 → Gemini 호출 → 결과 저장
├── dash_sales/               # Cloud Run Job: 매출 ETL
│   ├── Dockerfile
│   └── run_sales_etl.py      #   SAP 원본 → SALES_DAILY 요약
├── dash_ga4/                 # Cloud Run Job: GA4 ETL
│   ├── Dockerfile
│   └── run_ga_etl.py         #   GA4 이벤트 → 일별 성과 요약
├── sql/                      # BigQuery DDL
│   ├── ai_rag_schema.sql     #   RAG 3-레이어 테이블 스키마
│   ├── ai_job_queue.sql      #   비동기 잡 큐
│   └── ...
└── .streamlit/               # Streamlit 설정
```

---

## 전체 아키텍처

```
┌────────────────────────────────────────────────────────────────────┐
│                       Data Sources                                  │
│  SAP ZTIF9001 (매출 원본)    GA4 events_* (웹 분석)               │
└────────┬───────────────────────────────┬───────────────────────────┘
         │                               │
         ▼                               ▼
┌─────────────────┐            ┌─────────────────┐
│ dash_sales      │            │ dash_ga4         │
│ (Cloud Run Job) │            │ (Cloud Run Job)  │
│ 매출 ETL        │            │ GA4 ETL          │
└────────┬────────┘            └────────┬─────────┘
         │                               │
         ▼                               ▼
┌────────────────────────────────────────────────────────────────────┐
│                      BigQuery (DASHBOARD)                           │
│  SALES_DAILY · GA4_DAY_PERFORM · GA4_DAY_TRAFFIC · ...            │
│  AI_JOB_QUEUE · AI_REPORTS · AI_REPORT_CHUNKS · AI_REPORT_DATA    │
└────────────────────────────┬───────────────────────────────────────┘
                             │
                             ▼
┌────────────────────────────────────────────────────────────────────┐
│              Streamlit Dashboard (app.py / Cloud Run Service)       │
│                                                                    │
│  ┌──────┐ ┌────┐ ┌────┐ ┌──────┐ ┌────┐ ┌──────┐ ┌───────────┐ │
│  │연간  │ │주간│ │일간│ │판매상품│ │환불│ │이벤트│ │교재·GA·트렌드│ │
│  └──────┘ └────┘ └────┘ └──────┘ └────┘ └──────┘ └───────────┘ │
│                             │                                      │
│              ┌──────────────▼──────────────┐                      │
│              │    AI 리포트 탭              │                      │
│              │  ┌──────┐ ┌────┐ ┌─────┐   │                      │
│              │  │ 생성 │ │불러│ │챗봇 │   │                      │
│              │  │      │ │오기│ │(RAG)│   │                      │
│              │  └──┬───┘ └────┘ └──┬──┘   │                      │
│              └─────┼───────────────┼──────┘                      │
└────────────────────┼───────────────┼───────────────────────────────┘
                     │               │
                     ▼               ▼
┌─────────────────────┐   ┌──────────────────────────┐
│ dash_ai             │   │ Vertex AI Embedding       │
│ (Cloud Run Job)     │   │ text-multilingual-002     │
│ Gemini 3.5 Flash    │   │ (768차원 벡터)            │
│ Lite 호출           │   └──────────────────────────┘
└─────────────────────┘
```

---

## 핵심 구현 포인트

### 1. 비동기 AI 생성 — Streamlit 세션 보호

**문제**: Streamlit이 Gemini를 직접 호출하면 30~120초 블로킹 → WebSocket 타임아웃 → 세션 튕김

**해결 구조**:
```
[Streamlit]                          [Cloud Run Job: dash_ai]
     │                                       │
     ├─ "생성" 클릭                           │
     │   → BQ AI_JOB_QUEUE INSERT            │
     │     (status='pending')                │
     │                                       │
     ├─ Cloud Run Job 실행 트리거 ──────────→ │
     │                                       ├─ BQ에서 요청 읽기 (claim)
     │                                       ├─ Gemini API 호출
     │                                       ├─ 결과를 BQ에 저장
     │                                       │   (status='done')
     │                                       │
     ├─ 2초마다 BQ 폴링 ←────────────────────┘
     │   (status 체크)
     └─ 결과 표시
```

- Streamlit은 BQ에 INSERT 1건(1초) → 이후 가벼운 SELECT 폴링만
- 실제 Gemini 호출(최대 240초)은 독립 컨테이너에서 처리
- Job이 30초 이상 pending이면 재트리거 (이전 실행 종료 대비)
- `worker_id`로 동시 실행 시 중복 처리 방지

### 2. AI 리포트 생성 — 13개 섹션 구조화

**데이터 수집 → 프롬프트 조립 → 생성 → 파싱** 파이프라인:

1. 대시보드 각 탭의 데이터를 텍스트 요약으로 변환
   - 월별/주간/일간 매출, KPI 달성률, GA 채널 성과, 교재 순위, 이벤트 전환, 검색 트렌드, 환불/환급, 상품 매핑
2. `prompts/report_prompt.py`의 템플릿에 데이터 삽입
3. 13개 섹션(종합요약, 매출추이, 증가/감소 분석, 환불, 교재, GA, SA, 이벤트, Next Action 등)으로 구조화 출력 지시
4. 응답을 `## 섹션명` 기준으로 파싱 → 섹션별 UI 렌더링

**미완료 월 처리**: 기준일이 해당 월 마지막 날보다 이전이면 프롬프트에 "이 달은 진행 중이므로 KPI 미달로 분석하지 말 것" 경고를 동적 삽입.

### 3. 보관 시스템 — 리포트를 "조직 지식"으로 축적

보관 클릭 시 3개 테이블에 동시 적재:

| 레이어 | 테이블 | 용도 |
|--------|--------|------|
| 1. 헤더 | `AI_REPORTS` | 사람이 읽는 단위 (13개 섹션 + 담당자 의견) |
| 2. 청크 | `AI_REPORT_CHUNKS` | RAG 벡터 검색 단위 (임베딩 포함) |
| 3. 스냅샷 | `AI_REPORT_DATA` | 원본 DataFrame JSON (숫자 재검증용) |

- 같은 기간에 여러 건 보관 가능 (조회 조건이 달라도 OK)
- 같은 이름이면 해당 건만 교체 (upsert 방식)
- 메타데이터: 사업군, 기간, 기준일, 조회 조건(JSON), 생성자, 생성시각

---

## RAG (Retrieval-Augmented Generation) 구현 상세

### 전체 흐름

```
[리포트 보관]                              [챗봇 질문]
     │                                        │
     ▼                                        ▼
┌─────────────────┐                  ┌──────────────────┐
│ 청크 분할       │                  │ 질문 임베딩       │
│ (섹션별 + 데이터)│                  │ (RETRIEVAL_QUERY) │
└────────┬────────┘                  └────────┬─────────┘
         │                                    │
         ▼                                    ▼
┌─────────────────┐                  ┌──────────────────┐
│ 임베딩 생성     │                  │ VECTOR_SEARCH    │
│ text-multilingual│                  │ (COSINE, top_k=8)│
│ -embedding-002  │                  └────────┬─────────┘
│ (768차원)       │                           │
└────────┬────────┘                           ▼
         │                           ┌──────────────────┐
         ▼                           │ 근거 청크 수집    │
┌─────────────────┐                  │ + 메타데이터     │
│ AI_REPORT_CHUNKS│ ←────────────────┤                  │
│ (BQ 테이블)     │                  └────────┬─────────┘
└─────────────────┘                           │
                                              ▼
                                    ┌──────────────────┐
                                    │ Gemini 종합 답변  │
                                    │ (근거 인용 [1][2])│
                                    └──────────────────┘
```

### 청크 설계

리포트 보관 시 생성되는 청크 종류:

- **AI 분석 청크**: 13개 섹션 각각이 1개 청크 (담당자 의견 포함)
- **데이터 요약 청크**: 프롬프트에 사용된 원본 데이터 요약 (월별 매출, GA, 교재 순위 등)
- **컨텍스트 접두어**: 모든 청크에 `[사업군명 · 기간 · 리포트명]` 붙여 검색 시 맥락 제공

### 벡터 검색 (`_rag_search`)

```python
# BigQuery 네이티브 VECTOR_SEARCH — 별도 벡터DB 불필요
SELECT base.chunk_id, base.chunk_text, distance
FROM VECTOR_SEARCH(
  (SELECT * FROM AI_REPORT_CHUNKS WHERE bsark = @bsark AND ...),
  'embedding',
  (SELECT @query_embedding AS embedding),
  top_k => 8,
  distance_type => 'COSINE'
)
ORDER BY distance
```

- 사전 필터: 사업군, 기간 범위, 탭으로 검색 범위 축소 후 벡터 유사도 정렬
- 폴백: 임베딩 실패 시 `LIKE '%키워드%'` 텍스트 매칭 (graceful degradation)

### 답변 생성 (`_rag_answer`)

- 검색된 top-k 청크를 `[근거 1]`, `[근거 2]` 형태로 프롬프트에 주입
- "근거에 없는 내용은 추측하지 말 것" — 할루시네이션 방지
- "서로 다른 탭/기간의 데이터를 교차 비교할 것" — 크로스 분석 유도
- 답변에 `[1]`, `[2]` 인용 번호 → UI에서 근거 문서 펼쳐보기 지원

### 비교 리포트

- 현재 생성한 리포트 vs 과거 보관 리포트(최대 3건) 비교
- 5개 고정 섹션: 한눈에 보기, 개선 항목, 악화 항목, 반복 패턴, 지금 봐야 할 것
- 과거 담당자 의견도 AI에게 전달 → 맥락 있는 트렌드 분석

---

## 주요 BigQuery 테이블

| 테이블 | 용도 |
|--------|------|
| `SALES_DAILY` | 일별 매출 요약 (bsark × dt × level1~4 × lid) |
| `GA4_DAY_PERFORM` | GA4 일별 성과 (소스/매체별 세션·매출·전환) |
| `GA4_DAY_TRAFFIC` | GA4 일별 트래픽 (디바이스별) |
| `AI_JOB_QUEUE` | 비동기 AI 작업 큐 (pending → running → done/error) |
| `AI_REPORTS` | 보관된 리포트 헤더 + 섹션별 텍스트 + 담당자 의견 |
| `AI_REPORT_CHUNKS` | RAG 검색용 청크 + 768차원 임베딩 벡터 |
| `AI_REPORT_DATA` | 원본 DataFrame 스냅샷 (숫자 재검증용) |
| `BSARK_CONFIG` | 사업부 설정 (코드 수정 없이 사업군 추가 가능) |
| `AUTH_USERS` | 사용자 권한 관리 (OAuth + 사업군별 접근제어) |

---

## 사용 기술 스택

| 영역 | 기술 |
|------|------|
| Frontend | Streamlit (Python) |
| Backend/Hosting | Google Cloud Run (서비스 + Job 3개) |
| Database | BigQuery (분석 + 벡터 검색 + 잡 큐 겸용) |
| AI/LLM | Vertex AI — Gemini 3.5 Flash-Lite |
| Embedding | Vertex AI — text-multilingual-embedding-002 (768d) |
| Vector Search | BigQuery VECTOR_SEARCH (COSINE) |
| ETL | Python + BigQuery SQL (Cloud Run Job) |
| Auth | Google OAuth 2.0 + BQ 권한 테이블 |
| CI/CD | Cloud Build + gcloud CLI |
| Infra | Dockerfile × 4, Cloud Build, Cloud Scheduler |

---

## 대시보드 기능 요약

| 탭 | 기능 |
|----|------|
| 연간 | 월별 매출 테이블 + KPI 달성률 + 전년 동기 대비 |
| 주간 | 사내 주차 기준 매출 추이 + 전주/전년 비교 |
| 일간 | 일자별 매출 + 전년 요일동기 매칭 + 성장률 |
| 판매상품 | 카테고리별 매출/건수 비교 (올해 vs 전년) |
| 환불/환급 | 기간별 환불/환급 현황 + 사유 분석 |
| 이벤트 | 이벤트 참여 → 가입 → 구매 전환 퍼널 |
| 교재 순위 | 서점 교재 판매순위/판매지수 트래킹 |
| GA 성과 | 소스/매체별 세션·매출·전환 분석 |
| 검색 트렌드 | 네이버 DataLab API 연동 검색지수 추이 |
| 상품별 매핑 | 사용자 정의 상품 그룹 비교 |
| AI 리포트 | AI 생성 + 기존 불러오기 + RAG 챗봇 |


------------------------------------------------




## 문서

| 문서 | 내용 |
|------|------|
| [01. 매출 성과 대시보드](docs/01_매출_성과_대시보드.md) | 탭 구성, 매출 집계 기준, 사이드바 조건, 실행 방법 |
| [02. 검색어 트렌드](docs/02_검색어_트렌드.md) | 네이버 API 연동, 프리셋 저장, 키 발급 방법 |
| [03. AI 분석 리포트](docs/03_AI_분석_리포트.md) | Vertex AI 연결, 8개 섹션, 담당자 의견 |
| [04. GA ETL 잡 배포](docs/04_GA_ETL_잡_배포.md) | Cloud Run Job + Scheduler, 백필, 사업군 추가 |
| [05. 대시보드 배포](docs/05_대시보드_Cloud_Run_배포.md) |  |
| [06. 인증](docs/06_인증_및_사용자_권한.md) |  |
| [07. 유지보수](docs/07_유지보수_가이드.md) |  |


## 배포 & 접근 제어

앱은 **Cloud Run**에 배포되고, **Google OAuth 로그인 + 사업군별 접근 제어**로 보호됩니다.

- 배포 방법: [docs/05_대시보드_Cloud_Run_배포.md](docs/05_대시보드_Cloud_Run_배포.md)
- 인증 & 사용자 관리: [docs/06_인증_및_사용자_권한.md](docs/06_인증_및_사용자_권한.md)

**서비스 URL:** `https://sales-dashboard-397756181081.asia-northeast3.run.app`

빠른 재배포 (코드 수정 후):
```bash
export PROJECT_ID="ga4-bigquery-431807"
cd ~/sales-dashboard
gcloud builds submit --tag asia-northeast3-docker.pkg.dev/$PROJECT_ID/sales-dashboard/app:latest
gcloud run deploy sales-dashboard \
  --image=asia-northeast3-docker.pkg.dev/$PROJECT_ID/sales-dashboard/app:latest \
  --region=asia-northeast3 --platform=managed
```

## 데이터 소스 요약

| 데이터 | 위치 | 적재 |
|--------|------|------|
| 매출 (9001) | `HANABQ.ZTIF9001` | 하나DB → BQ (매일) |
| 이벤트 (9009) | `HANABQ.ZTIF9009` | 하나DB → BQ (매일) |
| 교재 순위 | `lab_asia.BSM_KWD` | 별도 크롤러 |
| GA 방문자/성과 | `DASHBOARD.GA4_DAY_TRAFFIC/PERFORM` | ETL 잡 (매일 07:00) |
| 트렌드 프리셋 | `DASHBOARD.TREND_PRESETS` | 앱에서 저장 |
| 사용자 권한 | `DASHBOARD.AUTH_USERS` | 사용자 관리 탭에서 저장 |
| 검색 트렌드 | 네이버 API (실시간) | 조회 시 호출 |
| AI 분석 | Vertex AI (실시간) | 버튼 클릭 시 호출 |

