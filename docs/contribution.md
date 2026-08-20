# My Contribution

## 1. Data Integration

- SAP / GA4 / 웹사이트 등 여러 시스템에 분산된 데이터를 하나의 분석 환경으로 통합
- BigQuery를 중심으로 매출·마케팅·콘텐츠 데이터를 분석 가능한 형태로 구성
- Airflow / DAG 기반 데이터 적재 및 반복 작업 자동화

## 2. Analytics Platform

- 통합된 데이터를 기반으로 업무용 성과 분석 대시보드 구현
- 매출, 상품, 이벤트, GA, 교재, 검색 트렌드 등 분석 영역 구성
- 사업부 / 기간 / 상품 / 분석 영역별 조회 조건 구성
- 전년 대비 변화와 주요 KPI를 조회할 수 있는 분석 흐름 구성

## 3. AI Analysis

- 대시보드에서 조회된 데이터를 분석 목적에 따라 선택적으로 AI 컨텍스트로 구성
- 분석 영역별 Prompt 로직 구성
- Gemini 기반 AI 성과 분석 리포트 생성
- 종합 요약과 Next Action까지 포함하는 구조 구성

## 4. Report Knowledge Base

- 생성된 AI 리포트와 담당자 검토 의견을 함께 저장
- 분석에 사용된 원본 데이터 요약 및 Snapshot 저장
- 리포트를 검색 단위 Chunk로 구성
- Embedding 생성 및 저장
- BigQuery `VECTOR_SEARCH` 기반 RAG 검색 구조 구현

## 5. Comparison

- 현재 리포트와 과거 보관 리포트 비교 기능 구현
- 개선 / 악화 항목과 반복 패턴 분석
- 과거 담당자의 의견까지 비교 컨텍스트에 포함
- 비교 결과를 다시 업무 판단에 활용할 수 있도록 구성

## 6. System Design Perspective

이 프로젝트에서의 핵심 작업은 개별 기능 구현보다
**분산된 데이터와 분석 결과를 하나의 업무 흐름으로 연결하는 것**이었습니다.

```text
Data Integration
      ↓
Unified Analytics
      ↓
AI Analysis
      ↓
Report Storage
      ↓
Comparison
      ↓
RAG Retrieval
```

## 공개 범위

> 🔒 실무 프로젝트로 실제 업무 데이터와 소스 코드는 보안상 공개하지 않습니다.
>
> 본 문서는 공개 가능한 범위에서 시스템 구조,
> 데이터 흐름, 분석 로직 및 구현 범위를 설명합니다.
