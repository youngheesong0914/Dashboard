# 동작 시나리오 (Acceptance Criteria) · v3.1

> v3.1 변경 (정합성 해소):
> - **A3**: 내비 전환은 SPA swap이 아니라 **JSP 페이지 이동**임을 명시 (아키텍처 v5와 정합. 화면 명세의 "swaps the main content" 문구는 와이어프레임 한정 표현으로 간주).
> - **D6**: 월 변경 시 **AI Insight는 재요청하지 않는다** (Refresh 버튼으로만 갱신). 종전 문구 삭제.
> - **D6**: Claim 월 선택기 범위를 Queue와 동일하게 **Mar–Jun 2026, 기본 Jun**으로 명시.
> - **§G 신설**: 차트 클릭-투-필터 상세 인터랙션을 본 문서에 통합.
> - B4 오타 수정.
> - **A5/B3/B5 (v3.1.1)**: 하드코딩돼 있던 건수("9", queueNote)를 응답값(`summary.total`) 기준으로 정정. Inbox 뱃지는 view와 무관하게 pending 건수 유지, 사이드바 뱃지는 로드 후 갱신.
> - **§H 신설 (v3.1.1)**: 와이어프레임 HTML 대비 의도된 변경 목록 + CLAIM 색상 미해결 항목 기록. Step 단계 명칭은 HTML 기준 **Claim Creation**으로 확정.

> v3 변경: 차트는 Vega-Lite로 렌더링하며, **차트 spec은 서버(외부 API, 목업 단계에서는 프록시 JSP)가 완성해 내려준다.** 프론트는 vegaEmbed로 렌더링만 하고 spec을 수정하지 않는다. 테이블은 종전대로 HTML + 클라이언트 정렬/필터.

> 기준: 화면 명세 (Hub Navigation · Screen Spec). 형식: "~하면 ~된다".
> 외부 API 미정 - 목업 JSON 기준으로 검증한다. 데이터·동작 충돌 시 본 문서 우선, 레이아웃·스타일은 화면 명세 우선.

## A. 허브 네비게이션 / 전역 셸

- A1. 루트(index.jsp) 접근 시 **Approval Queue 화면으로 이동**한다 (기본 화면).
- A2. 좌측 내비 레일(272px, sticky)에 브랜드 블록, "DASHBOARDS" 라벨, 메뉴 2개가 표시된다: `01 Sales-Deduction Approval Queue`, `02 Claim L/T Analysis`.
- A3. 메뉴 클릭 시 해당 화면 JSP(queue.jsp / claimLT.jsp)로 **페이지 이동**한다 (SPA 콘텐츠 swap 아님 - 페이지 내 데이터 갱신만 리로드 없이 수행).
- A4. 현재 페이지의 메뉴 항목은 활성 스타일(연회색 배경 `#f4f4f4` + 검정 좌측 바 + 검정 인덱스 칩)로 표시되고, 비활성 항목의 칩은 회색이다.
- A5. 01 메뉴에는 빨간 원형 카운트 뱃지가 표시된다. 건수는 **pending 건수(응답 `summary.total`)** 기준이며, 사이드바는 공통 include라 페이지 로드 후 Queue 응답값으로 갱신한다 (0건이면 뱃지 숨김. 목업 기준 9. Claim 페이지에서의 건수 조회 방식은 TBD - TODO 주석).
- A6. 헤더에는 아바타 "J"와 "Hello, Jane Doe" / "Welcome back - Sales · Approver"가 표시된다. **알림 벨은 없다.**
- A7. 내비 레일 하단에 "Additional dashboards will appear here as they launch." 문구가 표시된다.
- A8. 두 페이지 모두 sidebar.jsp / header.jsp 공통 include를 사용한다.

## B. Approval Queue - 헤더·토글·월 선택기

- B1. 페이지 로드 시 view=pending(Inbox), month=Jun 2026 기본값으로 자동 조회해 전 위젯을 렌더링한다.
- B2. 우측 상단에 "As of **2026-06-26 02:00:41** · {monthLabel}" 메타가 표시된다 (asOf는 응답값).
- B3. Inbox/All 세그먼트 토글: 활성 항목은 파란 배경(`#0381fe`)·흰 글자이고, Inbox에는 건수 뱃지가 붙는다. 건수는 **응답 `summary.total`(pending 기준)**이며 view=all로 전환해도 pending 건수를 유지한다 (목업 기준 [9]).
- B4. 토글 변경 시 **페이지 리로드 없이** 데이터가 재조회되어 전 위젯이 갱신된다. 차트는 새 spec으로 vegaEmbed를 재실행하고, 테이블은 tbody를 재생성한다.
- B5. 부제(queueNote)는 view에 따라 바뀐다 - pending: "{total} awaiting your approval" (응답값) / all: "Showing all items for {monthLabel} · monthly view".
- B6. 월 선택기(◀ {monthLabel}/{monthCode} ▶)는 Mar–Jun 2026 범위이며 기본 Jun이다.
- B7. view=pending일 때 월 선택기는 dim 처리(opacity .45)되고 동작하지 않는다. view=all일 때만 활성화된다.
- B8. view=all 상태에서 월을 변경하면 해당 월 데이터로 즉시 재조회된다. 범위 밖(◀ at Mar, ▶ at Jun)으로는 이동하지 않는다.

## C. Approval Queue - 배너·차트·목록

- C1. 리스크 배너(핑크 bg)에 빨간 펄스 점 + "**5** pending items carry risk flags. Review these first." + 우측 "View risk items" 버튼이 표시된다 (건수는 응답값).
- C2. "View risk items" 클릭 시 하단 목록이 Risk only 필터로 전환된다 (상세: §G3).
- C3. **Pending by stage** 차트: 응답의 `widgets.byStage` spec을 vegaEmbed로 렌더링하면 세로 스택 바(하단 risk 빨강 `#ef3434`·흰 수치 + 상단 clean 연파랑 `#bcd9ff`·파란 수치, X 라벨 "스테이지명 + {total} items")가 그려진다. 스택 구성·색·라벨은 spec이 결정하며 프론트는 spec을 수정하지 않는다.
- C4. by stage 바의 risk/clean 부분을 클릭하면 Vega 클릭 이벤트가 발생하고, 프론트 JS가 이를 수신해 하단 목록을 해당 스테이지×상태로 필터한다 (재조회 없음). **상세 동작·이벤트 필드·토글 해제 규칙은 §G를 따른다.**
- C5. **Risk by type** 차트: 응답의 `widgets.riskByType` spec을 vegaEmbed로 렌더링하면 리스크 유형별 가로 스택 바(스테이지 색 세그먼트 + 세그먼트 내 건수 + 우측 합계 + 스테이지 범례)가 그려진다.
- C6. Risk by type의 risk×stage 집계는 **서버(spec 조립 시점)에서 수행**되어 spec 안에 포함된다. 클라이언트 집계는 하지 않는다.
- C7. 색 세그먼트를 클릭하면 Vega 클릭 이벤트를 수신해 목록이 해당 스테이지+리스크 유형으로 필터된다 (재조회 없음). **상세는 §G2.**
- C8. 목록 헤더: "Pending items" · "{n} shown · **{r} risk**" · 우측 세그먼트 [All | Risk only | Clean only] + "All stages ▾" select. 두 컨트롤 모두 **클라이언트 필터**이며 조합 적용된다.
- C9. 목록 컬럼: Condition ID(모노스페이스), Stage(필: CREATE 파랑/CHANGE 청록/CLAIM 보라 계열), Risk type(리스크당 빨간 칩, 없으면 회색 "Clean"), Checked at, Open report(우측 정렬 pill 버튼).
- C10. 리스크 행은 bg `#fffafa` + 좌측 inset 빨간 바(3px)로 구분된다.
- C11. "Open report" 클릭 동작은 미정 - 버튼과 핸들러 지점만 구현하고 TODO 주석을 남긴다.

## D. Claim L/T Analysis

- D1. 페이지 로드 시 month=Jun 2026 기본값으로 대시보드 데이터와 AI Insight를 조회해 렌더링한다.
- D2. **AI Insight 카드**(파란 bg): "AI" 칩 + 제목 + "LLM-generated summary" + 우측 "↻ Refresh" 버튼, 본문에 요약 문단이 표시된다.
- D3. AI Insight는 **페이지 로드 시 1회 + Refresh 클릭 시에만** 요청된다. Refresh 클릭 시 **AI Insight만** 재요청되어 갱신된다 (대시보드 본 데이터 재조회 없음). 요청 중 버튼은 비활성/로딩 표시.
- D4. AI 요약 생성 주체는 TBD - 목업 텍스트를 반환하는 proxyClaimInsight.jsp로 구현하고 TODO 주석을 남긴다.
- D5. 우측에 "Data is provided for the most recent settlement month." 문구와 월 선택기가 표시된다.
- D6. 월 선택기 범위는 **Mar–Jun 2026, 기본 Jun** (Queue와 동일)이며 범위 밖으로 이동하지 않는다. 월 변경(◀▶) 시 **즉시**(Apply 없음) 대시보드 데이터가 재조회되어 KPI·차트·테이블이 리로드 없이 갱신된다. **AI Insight는 월 변경 시 재요청하지 않는다** (Refresh 버튼으로만 갱신).
- D7. KPI 스트립: "Claim L/T (Days) **122.4**" | 구분선 | "No. Claims (EA) **3,329**" (응답값).
- D8. **Monthly Trend**: 응답의 `widgets.monthlyTrend` spec을 렌더링하면 12개월 콤보(파란 막대=Number of Claims, 주황 선+값 라벨=L/T Avg Days, 우상단 범례)가 그려진다.
- D9. **by Product**: 응답의 `widgets.byProduct` spec을 렌더링하면 가로 바(블루 셰이드, CTV만 앰버) + 우측 값이 Mobile 194.9 ~ CTV 93.0 순으로 그려진다.
- D10. **Step**: 라벨/값 리스트 - 1~4단계 + Customer Delay/Samsung Delay(소계, muted) + **Sum**(bold). Sum은 KPI L/T와 일치한다.
- D11. **by Customer** 테이블: 헤더에 빨간 뱃지 "Over Company Avg · {avg} days" + 우측 "↓ Raw Data Download" 버튼, 캡션에 안내 문구가 표시된다.
- D12. by Customer에는 **회사 평균 초과 고객만** 표시된다.
- D13. 컬럼: Customer | No. Claims | L/T Avg Days(블루 하이라이트 컬럼, 값은 빨강) | 1 | 2 | Customer Delay(1+2)(앰버) | 3 | 4 | Samsung Delay(3+4)(앰버). Delay 컬럼은 클라이언트에서 s1+s2 / s3+s4로 계산한다.
- D14. 모든 컬럼 헤더에 정렬 어포던스(⇅)가 있고, 클릭 시 오름/내림이 토글되며 **클라이언트에서** 정렬된다 (재조회 없음).
- D15. 테이블은 min-width ~1040px로 가로 스크롤된다.
- D16. Raw Data Download 클릭 시 현재 월 기준 다운로드가 시작된다 - 실제 파일 생성은 TBD, 호출 지점만 구현하고 TODO 주석.

## E. 오류·경계 상황

- E1. 데이터 요청 실패(네트워크/5xx) 시 위젯 영역에 오류 메시지 + "다시 시도" 버튼이 표시되고 기존 화면이 깨지지 않는다.
- E2. 빈 데이터(0건) 시 각 위젯에 "데이터 없음" 상태가 표시된다 (빈 차트·NaN 노출 금지).
- E3. 월 선택기·토글을 빠르게 연타해도 진행 중 요청이 중단(abort)되고 **마지막 요청의 응답만** 반영된다.
- E4. 미로그인 데이터 요청 시 401을 받고 로그인 안내로 연결된다 - 세션 연동 방식 미정, 체크 지점만 구현하고 TODO.
- E5. 모든 프록시 JSP는 **spec 형태의 목업 JSON**으로 동작하며(외부 API가 반환할 형태와 동일 구조), API 확정 시 프록시 내부 호출부만 교체하면 되도록 호출 코드를 한 곳(apiCommon.jsp)에 모은다.

## F. 비기능 요건

- F1. 데이터 조회 중에도 사용자 조작이 막히지 않는다 (비동기).
- F2. 신규 대시보드 추가 = 화면 JSP 1 + 프록시 JSP 1 + 사이드바 메뉴 1줄 패턴을 유지한다.
- F3. 정상 시나리오에서 브라우저 콘솔 에러가 없어야 한다.
- F4. 스타일은 화면 명세 Design tokens(Manrope/Noto Sans KR, radius 8px, 플랫·섀도 없음, 지정 색상표)를 따른다. 차트 내부 스타일은 spec이 결정한다.
- F5. 프론트는 수신한 Vega spec을 수정하지 않는다 (컨테이너 크기 등 vegaEmbed 옵션 수준만 허용).

---

## G. 차트 클릭-투-필터 상세 (Approval Queue)

> 대상: Approval Queue의 두 차트(Pending by stage, Risk by type) 클릭 → Pending items 테이블 필터.
> 원칙: spec의 클릭 대상(datum 필드) 정의는 **API(spec) 몫**, 이벤트 수신·필터 적용은 **프론트 몫** (아키텍처 v5 §5). 필터는 전부 클라이언트 처리, 재조회 없음.

### G1. 필터 상태 모델 (단일 소스)

```js
filterState = {
  status:   'all' | 'risk' | 'clean',   // 세그먼트 [All | Risk only | Clean only]
  stage:    null | 'CREATE' | 'CHANGE' | 'CLAIM',   // "All stages ▾" select
  riskType: null | string               // 차트(Risk by type) 클릭으로만 설정됨
}
```

- 모든 진입점(세그먼트, stages select, 차트 클릭, View risk items)은 **이 상태만 갱신**하고, 렌더는 단일 함수 `applyFilter()`가 수행한다: tbody 재생성 + 헤더 카운트 "{n} shown · {r} risk" 재계산 + 컨트롤 UI 동기화.
- 행 매칭 (AND 조합):
  - status: `risk` → `item.risks.length > 0` / `clean` → `=== 0` / `all` → 통과
  - stage: `item.stage === filterState.stage` (null이면 통과)
  - riskType: `item.risks.includes(filterState.riskType)` (null이면 통과)

### G2. 차트 클릭 동작

#### G2.1 공통 (이벤트 수신)

```js
vegaEmbed(el, spec, {actions: false}).then(result => {
  result.view.addEventListener('click', (event, item) => {
    if (!item || !item.datum) return;   // 여백·축·범례 클릭은 무시
    // datum → filterState 갱신 → applyFilter()
  });
});
```

- 재조회(토글/월 변경)로 **spec을 재embed할 때마다 리스너를 다시 등록**한다 (이전 view는 `result.view.finalize()`로 해제).
- 클릭 가능한 mark에는 spec에서 `"cursor": "pointer"`를 지정한다 (API 몫).

#### G2.2 Pending by stage (세로 스택 바)

- 클릭 대상: 바의 risk 세그먼트 또는 clean 세그먼트.
- datum 계약: `{ stage: 'CREATE'|'CHANGE'|'CLAIM', band: 'risk'|'clean', count: number }`
- 동작:
  - `filterState.stage = datum.stage`
  - `filterState.status = (datum.band === 'risk') ? 'risk' : 'clean'`
  - `filterState.riskType = null` (기존 리스크 유형 필터 해제)
- **토글 해제**: 직전 클릭과 동일한 (stage, band)를 다시 클릭하면 `{status:'all', stage:null, riskType:null}`로 초기화.
- 예: CHANGE 바의 빨간(risk) 부분 클릭 → 목록에 CHANGE×risk 1건(A2S7B0C26610620)만 표시, 헤더 "1 shown · 1 risk".

#### G2.3 Risk by type (가로 스택 바)

- 클릭 대상: 리스크 유형 행 안의 스테이지 색 세그먼트.
- datum 계약: `{ riskType: string, stage: 'CREATE'|'CHANGE'|'CLAIM', count: number }`
- 동작: `filterState = { status:'risk', stage: datum.stage, riskType: datum.riskType }`
- **토글 해제**: 동일 (riskType, stage) 재클릭 시 전체 초기화 (G2.2와 동일 규칙).
- 예: "Budget exceeded" 행의 CLAIM 세그먼트 클릭 → CLAIM이면서 risks에 Budget exceeded를 포함한 2건(A2S330D26784516, A2S560A26771330)만 표시.

### G3. 다른 컨트롤과의 상호작용 규칙

| 조작 | filterState 변화 |
|---|---|
| 세그먼트 [All/Risk only/Clean only] 클릭 | `status`만 변경, `riskType = null` (stage 유지) |
| "All stages ▾" select 변경 | `stage`만 변경, `riskType = null` (status 유지) |
| "View risk items" (배너) | `{status:'risk', stage:null, riskType:null}` + 목록으로 스크롤 |
| Inbox/All 토글, 월 변경 (재조회) | **전체 초기화** `{all, null, null}` 후 새 데이터 렌더 |

- 요지: `riskType`은 차트 클릭으로만 설정되고, **수동 컨트롤을 하나라도 만지면 해제**된다 (사용자가 보이지 않는 필터에 갇히는 것 방지).

### G4. UI 피드백

- 차트 클릭 후 컨트롤 동기화: 세그먼트 활성 표시 = `status`, stages select 값 = `stage ?? 'All stages'`.
- `riskType` 필터가 활성일 때는 컨트롤로 표현이 안 되므로, 목록 헤더에 **제거 가능한 필터 칩**을 표시한다: `[CLAIM × Budget exceeded ✕]` - ✕ 클릭 시 `riskType = null`.
- 차트 내 선택 세그먼트 하이라이트(비선택 dim)는 spec의 selection 정의가 필요 - **API 명세 협의 항목** (미지원 시 필터 칩만으로 충분).
- 필터 결과 0건: tbody에 "No items match the current filter" + "Clear filters" 링크 (`{all, null, null}` 초기화).

### G5. API 명세 반영 사항 (datum 계약)

프록시 목업 spec도 아래와 동일한 필드명으로 조립한다. API 확정 시 이 표를 그대로 명세에 포함할 것.

| 차트 | mark당 datum 필수 필드 | 값 |
|---|---|---|
| `widgets.byStage` | `stage` | `CREATE` / `CHANGE` / `CLAIM` |
| | `band` | `risk` / `clean` |
| | `count` | 세그먼트 건수 |
| `widgets.riskByType` | `riskType` | `Budget exceeded` 등 - `tables.items[].risks`의 문자열과 **정확히 일치**해야 매칭 가능 |
| | `stage`, `count` | 위와 동일 |

- 필드명이 달라지면 프론트 핸들러의 매핑 한 곳만 수정하도록, datum → filterState 변환을 어댑터 함수 하나로 격리한다 (TODO 주석).

---

## 미정 사항 (TODO로 표시)

| 항목 | 상태 |
|---|---|
| 외부 데이터 API 엔드포인트·인증 | 미정 - 목업 JSON (단, **spec까지 API가 반환하기로 확정**) |
| 차트 클릭 이벤트 필드 정의 (stage/risk 등) | **잠정 정의됨** - §G5의 datum 계약을 API 명세에 반영해 확정할 것 |
| AI Insight 요약 생성 주체 | 미정 - 목업 텍스트 |
| Open report 클릭 동작 | 미정 |
| Raw Data Download 파일 생성 | 미정 |
| 로그인/세션 연동 방식 | 미정 |
| CLAIM 스테이지 색상 (보라 vs 주황) | 미정 - §H 참조, 확정 전까지 화면 명세(보라) 적용 |

## 타 문서 반영 필요 사항 (한 줄 패치)

- **화면 명세 · Global shell**: "Clicking a nav item swaps the main content" → "Clicking a nav item navigates to that page (separate JSP)". `State: nav ∈ {queue, claim}` 문구 삭제.
- **화면 명세 · Page 2 Step 카드**: "3. I/V Receipt ~ SAP Creation / 4. SAP Creation ~ End of Approval" → "~ **Claim Creation** / **Claim Creation** ~"로 정정. 와이어프레임 HTML은 Step 카드·by Customer 컬럼 모두 **Claim Creation**을 사용 - 같은 문서 안 by Customer 컬럼과도 이제 일치.
- **아키텍처 v5 · §4.2**: Claim 월 선택기 범위 "Mar–Jun 2026, 기본 Jun" 명시. AI Insight 트리거는 §4.3(로드 1회 + Refresh)이 정본 - 월 변경 트리거 없음.

## H. 와이어프레임(HTML) 대비 차이 - 구현 시 문서 기준을 따를 것

현재 와이어프레임 DC는 본 문서 이전 이터레이션이다. 아래 항목은 **문서가 정본**이며, 와이어프레임을 그대로 옮기면 안 된다.

| # | 와이어프레임 HTML | 본 문서 (정본) |
|---|---|---|
| 1 | 상단 헤더 탭 4개 (01 Approval Review · 02 Claim L/T · 03 Overdue Conditions · 04 Target Plan Budget) | **좌측 내비 레일 272px, 메뉴 2개** (01 Sales-Deduction Approval Queue · 02 Claim L/T Analysis). 03/04 화면은 스코프 제외 |
| 2 | H1 "Approval Review", 헤더 서브 "Approver Tracker Hub", 우측 "Jane Doe"만 | H1 "Sales-Deduction Approval Queue", "Approver Dashboard Hub", "Hello, Jane Doe / Welcome back - Sales · Approver", 레일 하단 안내 문구(A7) |
| 3 | Company/BusArea(/Product) 필터 + **Apply 버튼** | 필터·Apply 없음 - 월 선택기 **즉시 반영** (B8/D6) |
| 4 | Inbox/All 토글 활성 = 검정 bg | 활성 = **파랑 `#0381fe`** (B3) |
| 5 | "As of ..." 메타 없음 | B2 표시 |
| 6 | 월 선택기 항상 dim(정적) | pending일 때만 dim, all일 때 활성 (B7) |
| 7 | AI Insight 카드에 Refresh 버튼 없음 | ↻ Refresh 있음 (D2/D3), 우측 "Data is provided..." 문구 (D5) |
| 8 | 목록 컨트롤에 "All risk types ▾" select 있음 | **제거** - 리스크 유형 필터는 차트 클릭 + 필터 칩(§G4)으로만 진입 (C8) |

**확인 필요 (미해결)**: CLAIM 스테이지 색상. 와이어프레임 HTML은 CLAIM을 주황(`#e8863c`, pill은 앰버 `#fdf6ee/#b5701f`)으로 쓰는데, 화면 명세는 보라(`#6b4fc0`, pill `#ecedfb/#4b4fc0`)로 정의한다. 주황은 Monthly Trend의 L/T 선 색과 겹치므로 보라로 바꾼 것으로 추정되나 **디자인 확정 필요** - 확정 전까지 화면 명세(보라)를 따른다.
