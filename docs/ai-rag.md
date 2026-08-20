# AI / RAG

## 1. AI Analysis

대시보드에서 조회한 데이터를 그대로 모델에 넘기지 않고,
분석 목적에 맞는 데이터 블록을 구성해 AI 컨텍스트로 전달합니다.

컨텍스트에 포함될 수 있는 데이터:

- KPI / 매출
- 상품
- GA 성과
- 이벤트
- 교재
- 검색 트렌드
- 환불 / 환급
- 상품 매핑
- 주간 / 일간 매출

## 2. 선택적 섹션 생성

사용자가 선택한 분석 영역에 따라 생성할 리포트 섹션을 결정합니다.

```text
사용자 선택
    ↓
분석 영역 결정
    ↓
필요 데이터만 Context 구성
    ↓
Gemini
    ↓
Structured AI Report
```

항상 포함되는 영역:

- 종합 요약
- Next Action

조건부 분석 영역:

- 매출 추이
- 매출 증가 / 감소
- 환불 / 환급
- 상품 그룹 비교
- 교재 순위
- 검색 트렌드
- GA 채널 효율
- 이벤트 전환
- 검색 트렌드 × 교재 순위

## 3. Data Completeness Handling

진행 중인 기간은 데이터가 아직 완성되지 않을 수 있기 때문에,
최종 KPI 미달로 오해하지 않도록 별도 경고 컨텍스트를 생성합니다.

```text
진행 중인 기간
      ↓
불완전 데이터 경고
      ↓
AI Prompt에 포함
      ↓
확정되지 않은 수치를 고려한 분석
```

## 4. Report Storage

### `AI_REPORTS`

리포트 섹션과 담당자 의견, 리포트 메타데이터를 저장합니다.

### `AI_REPORT_CHUNKS`

RAG 검색을 위한 텍스트 Chunk와 Embedding을 저장합니다.

### `AI_REPORT_DATA`

분석 당시 사용한 데이터의 Snapshot을 저장합니다.

## 5. RAG Corpus

리포트 저장 시 AI 분석 결과, 담당자 의견, 분석 근거가 된 데이터 요약을 검색 가능한 단위로 구성합니다.

```text
AI Report Section
        +
Manager Opinion
        +
Source Data Summary
        ↓
      Chunk
        ↓
    Embedding
        ↓
AI_REPORT_CHUNKS
```

이 구조를 통해 과거 분석 결과뿐 아니라 당시 담당자가 남긴 해석과 분석 근거까지 검색 컨텍스트로 활용할 수 있습니다.

## 6. Report Comparison

현재 리포트와 과거 리포트를 비교할 때 다음 정보를 함께 사용합니다.

```text
Current Report
      +
Past Reports
      +
Past Manager Opinions
      ↓
Comparison Prompt
      ↓
Gemini
      ↓
Comparison Result
```

비교 결과:

1. 한눈에 보기
2. 개선된 항목
3. 악화된 항목
4. 반복되는 패턴
5. 지금 봐야 할 것

## 7. Vector Search

저장된 Chunk의 Embedding과 사용자 질문의 Embedding을 비교하여 관련 Chunk를 검색합니다.

```text
Question
   ↓
Query Embedding
   ↓
BigQuery VECTOR_SEARCH
   ↓
Top-K Chunks
   ↓
Gemini
   ↓
Answer
```

별도의 Vector Database 없이 BigQuery의 `VECTOR_SEARCH`를 사용합니다.

## 8. Design Principle

**분산된 데이터를 먼저 통합하고 → 분석 가능한 형태로 가공하고 → AI가 분석하고 → 사람이 검토하고 → 결과를 축적하고 → 비교와 검색에 재사용하는 것**

이 프로젝트는 이 흐름을 하나의 업무 시스템으로 연결합니다.

```text
Data Integration
      ↓
Analytics
      ↓
AI Analysis
      ↓
Human Review
      ↓
Knowledge Accumulation
      ↓
Comparison / Retrieval
```
