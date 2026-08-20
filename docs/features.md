# Features

## 1. 매출 성과 분석

![Annual Sales](images/screenshot_annual.png)

- 연간 / 주간 / 일간 조회
- KPI 달성률 계산
- 전년 동기 대비 증감률
- LEVEL1~4 Drill-down
- 상품코드 기반 필터링

## 2. 이벤트 전환 분석

![Event](images/screenshot_event.png)

- 무료 이벤트 → 회원가입 → 유료 구매 퍼널
- 이벤트별 매출 기여도
- 연간 / 주간 / 일간 추이

## 3. GA 마케팅 성과

![GA](images/screenshot_ga.png)

- Source / Medium별 성과
- 세션 / 매출 / 전환율
- 네이버 SA 채널 성과
- 캠페인 필터링

## 4. AI 리포트 생성

![AI Report](images/screenshot_ai_report.png)

사용자가 선택한 분석 영역만 AI에 전달하여 필요한 리포트 섹션을 생성합니다.

주요 분석 영역:

- 매출 추이
- 매출 증가 / 감소
- 환불 / 환급
- 상품 그룹 비교
- 교재 순위
- 검색 트렌드
- GA 성과
- 이벤트 전환
- 검색 트렌드 × 교재 순위
- 종합 요약
- Next Action

### 주요 특징

- 선택된 분석 영역만 프롬프트에 포함
- 진행 중인 기간의 불완전 데이터에 대한 주의사항 처리
- 섹션별 결과를 카드형 UI로 표시
- 담당자 검토 의견 입력 지원

## 5. 리포트 저장 & 재활용

생성된 리포트를 비교 분석과 RAG 검색에 활용할 수 있도록 저장합니다.

저장 데이터:

- AI 분석 내용
- 담당자 검토 의견
- 리포트 메타데이터
- 분석 당시 조건
- 분석 데이터 요약
- 원본 데이터 Snapshot

## 6. 과거 리포트 비교

![Comparison](images/screenshot_comparison.png)

현재 리포트와 과거 보관 리포트를 선택하여 시간에 따른 변화를 비교합니다.

비교 결과:

- 한눈에 보기
- 개선된 항목
- 악화된 항목
- 반복되는 패턴
- 지금 봐야 할 것

과거 리포트에 남겨진 담당자 의견도 비교 컨텍스트에 포함합니다.

## 7. RAG 챗봇

![Chatbot](images/screenshot_chatbot.png)

보관된 리포트와 분석 데이터를 검색 가능한 Chunk로 구성한 뒤 Embedding 기반 검색을 수행합니다.

```text
User Question
      ↓
Question Embedding
      ↓
BigQuery VECTOR_SEARCH
      ↓
Top-K Relevant Chunks
      ↓
Gemini
      ↓
Grounded Answer
```

- 자연어 질문
- Vector Search 기반 관련 리포트 검색
- 검색 근거를 기반으로 한 답변
- 과거 담당자 의견 활용
