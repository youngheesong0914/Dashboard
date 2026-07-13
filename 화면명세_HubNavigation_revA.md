# Sales-Deduction AI Agent — Hub Navigation · Screen Spec (rev A)

Mid-fi wireframe spec for developer handoff. Two dashboards behind a shared left nav rail. Desktop web app. All copy in English.

> rev A (정합성 패치, 동작 시나리오 v3.1과 정합):
> - Global shell: 내비 클릭 = 페이지 이동으로 문구 수정 (`State: nav` 삭제).
> - Page 2 Step 카드: "SAP Creation" → "**Claim Creation**" 정정 (by Customer 컬럼·와이어프레임 HTML과 통일).
> - CLAIM 색상(보라)은 와이어프레임 HTML(주황)과 다름 - 의도된 변경으로 잠정 유지, 확정 필요 (AC §H 참조).

## Design tokens

- **Canvas:** `#f7f7f7` (page) / `#ffffff` (cards, nav)
- **Text:** `#000` primary, `#707070` secondary, `#909090` muted
- **Hairline / border:** `#dddddd`; row dividers `#f2f2f2`; card radius `8px`
- **Accent (action / link):** `#0381fe` blue; hyperlinks `#0072de`
- **Danger / risk:** `#ef3434`; risk row bg `#fffafa`; risk banner bg `#fdecec` / border `#f6c7c7`
- **Stage colors:** CREATE `#0381fe`, CHANGE `#0c8fa0`, CLAIM `#6b4fc0`
  - Stage pills (list): CREATE `bg #e3edfd / text #0067d6`, CHANGE `bg #e1f2f5 / text #0c8fa0`, CLAIM `bg #ecedfb / text #4b4fc0`
- **Fonts:** Display = Manrope (700/800, headings, numbers); Body/UI = Noto Sans KR. Bold (700) dominates.
- **Shadows:** none (flat). Depth via bg shift + hairlines.

## Global shell

- **Left nav rail** (272px, sticky, full height, white, right border):
  - Brand block: black rounded square "S" + "Sales-Deduction AI Agent" / "Approver Dashboard Hub"
  - Section label "DASHBOARDS", then nav items. Active item: light-grey bg `#f4f4f4`, black left bar, black index chip. Inactive: grey chip.
  - Items: `01 Sales-Deduction Approval Queue` (red count badge, value from response), `02 Claim L/T Analysis`
  - Footer note: "Additional dashboards will appear here as they launch."
- **Main header** (both pages): avatar "J" + "Hello, Jane Doe" / "Welcome back — Sales · Approver". (No notification bell.)
- Clicking a nav item **navigates to that page** (separate JSP per dashboard).

---

## Page 1 — Sales-Deduction Approval Queue

### Header row
- H1 "Sales-Deduction Approval Queue"; subtitle `queueNote`.
- Top-right meta: "As of **2026-06-26 02:00:41** · {monthLabel}".
- Toggle (pill segmented): **Inbox** [{total}] / **All**. Active = blue `#0381fe` bg, white text. State `view ∈ {pending, all}`.
  - `queueNote` = pending: "{total} awaiting your approval"; all: "Showing all items for {monthLabel} · monthly view".
- **Month selector** (right): ◀ / ▶ around "{monthLabel}" + "{monthCode}" small. Months: Mar–Jun 2026 (`monthIdx`, default Jun). Dimmed (opacity .45) when view=pending, full when view=all. *Behavior intent: selecting All loads data by month.*

### Risk banner
Full-width pink banner: red pulse dot + "**5** pending items carry risk flags. Review these first." + right "View risk items" outline button.

### Charts row (2 cols)
1. **Pending by stage** — vertical stacked bars, one per stage (CREATE/CHANGE/CLAIM). Each bar = clean (light blue `#bcd9ff`, count in `#1f63c4`) stacked over risk (red `#ef3434`, white count). Bar height ∝ stage total / max total. X-label: stage name + "{total} items". Legend: Risk / Clean. Caption: "Click the risk / clean part of a bar to filter the list below."
   - Data: CREATE total 3 (risk 1), CHANGE total 2 (risk 2), CLAIM total 4 (risk 2).
2. **Risk by type** — horizontal **stacked bars broken down by stage**. One row per risk type; segments colored by stage with count inside each segment; grand total at right; stage legend below. Segment width ∝ count / max-row-total. Caption: "Click a colored segment to filter by that stage and risk type."
   - Derived from the pending rows (below), tallied by `risk × stage`:
     - Budget exceeded → CHANGE 1, CLAIM 2 (total 3)
     - Post-approval → CHANGE 1, CLAIM 1 (total 2)
     - Deal Sheet Exception → CREATE 1 (total 1)

### Pending items table
- Header: "Pending items" · "9 shown · **5 risk**" · right controls: segmented [All | Risk only | Clean only] + "All stages ▾" select.
- Columns: **Condition ID** (mono), **Stage** (pill), **Risk type** (red chips per risk, or grey "Clean"), **Checked at**, **Report** ("Open report" pill button, right-aligned).
- Risk rows: bg `#fffafa` + inset red left bar (`box-shadow: inset 3px 0 0 #ef3434`).
- Row data (`id / stage / risks[] / when`):
  | id | stage | risks | when |
  |---|---|---|---|
  | A2S910E26540159 | CLAIM | — | 06-26 16:34 |
  | A2S7B0C26610620 | CHANGE | Budget exceeded | 06-26 16:06 |
  | A2S330D26987267 | CREATE | — | 06-25 15:50 |
  | A2S330D26784516 | CLAIM | Post-approval, Budget exceeded | 06-25 08:19 |
  | A2S560A26699512 | CREATE | Deal Sheet Exception | 06-24 16:38 |
  | A2S7B0C26436239 | CLAIM | — | 06-24 11:20 |
  | A2S910E26512044 | CHANGE | Post-approval | 06-23 09:12 |
  | A2S330D26689026 | CREATE | — | 06-23 08:40 |
  | A2S560A26771330 | CLAIM | Budget exceeded | 06-22 17:05 |

---

## Page 2 — Claim L/T Analysis

### Header row
- H1 "Claim L/T Analysis" only (no subtitle).

### AI insight + month selector row
- **AI Insight card** (blue `bg #eef4ff / border #cfe0fb`, flex-grow): "AI" chip + "AI Insight" + "LLM-generated summary" + right "↻ Refresh" outline pill. Body = `aiSummary` (LLM-style paragraph).
  - `aiSummary`: "For the {monthLabel} settlement, the average Claim L/T is 122.4 days — a 0.6-day improvement over the prior month (123.0 days). Exertis Ireland Estore (799.0 days) and the Mobile division (193.8 days) far exceed the company average and should be reviewed first. About 68% of total delay occurs in the Customer Delay (Promo End ~ I/V) segment."
- **Right column:** note "Data is provided for the most recent settlement month." + same month selector (◀ {monthLabel} / {monthCode} ▶), months Mar–Jun 2026, default Jun.

### KPI strip (one row)
"Claim L/T (Days) **122.4**" | divider | "No. Claims (EA) **3,329**".

### Charts row (3 cols: trend 1.5fr / product 270px / step 270px)
1. **Monthly Trend** — combo chart, 12 months '25.07 → '26.06. Blue bars = Number of Claims; orange polyline + labels = L/T Avg Days. Legend top-right.
   - Data (`short / claims / avg`): '25.07 2734/101.0, '25.08 2856/94.0, '25.09 2790/94.3, '25.10 2529/134.3, '25.11 2648/89.6, '25.12 3068/106.9, '26.01 3141/97.0, '26.02 2527/91.2, '26.03 3321/109.3, '26.04 2031/105.9, '26.05 2372/123.0, '26.06 3329/122.4.
2. **by Product** (L/T Avg Days) — horizontal bars, blue shades (+ CTV amber `#f2d98c`): Mobile 194.9, Computer 185.9, Memory 164.1, Brown Goods 127.2, White Goods 106.8, CTV 93.0.
3. **Step** — label / value list, L/T Avg Days. Sub-rows muted, total row bold:
   - 1. Promo End ~ I/V Issue 72.7
   - 2. I/V Issue ~ I/V Receipt 4.4
   - *Customer Delay* 77.1
   - 3. I/V Receipt ~ Claim Creation 18.9
   - 4. Claim Creation ~ End of Approval 26.4
   - *Samsung Delay* 45.2
   - **Sum 122.4**

### by Customer (full-width table)
- Header: "by Customer" + red badge "Over Company Avg · 122.4 days" + right "↓ Raw Data Download" outline button.
- Caption: "Showing only customers whose L/T Avg Days exceeds the company average (122.4 days) · Click a column header to sort ascending/descending". Each header shows a sort affordance (⇅).
- Columns: **Customer** (left) | No. Claims | **L/T Avg Days** (highlighted blue col, value red) | 1. Promo End ~ I/V Issue | 2. I/V Issue ~ I/V Receipt | **Customer Delay (1+2)** (amber-highlighted, = s1+s2) | 3. I/V Receipt ~ Claim Creation | 4. Claim Creation ~ End of Approval | **Samsung Delay (3+4)** (amber-highlighted, = s3+s4). Horizontally scrollable (min-width ~1040px).
- Rows (`name / claims / lt / s1 / s2 / s3 / s4`; cust=s1+s2, sam=s3+s4):
  | Customer | Claims | L/T | s1 | s2 | s3 | s4 |
  |---|--|--|--|--|--|--|
  | 2430031 Exertis Ireland Estore | 140 | 799.0 | 771.3 | 2.5 | 5.6 | 19.6 |
  | 2015695 Shop Direct Home Shopping | 128 | 186.6 | 115.8 | 2.3 | 10.9 | 57.6 |
  | 4244888 TD Synnex UK Limited | 92 | 180.5 | 130.4 | 2.5 | 8.9 | 38.7 |
  | 4531120 Partner Retail Services Ltd | 67 | 166.3 | 67.2 | 5.6 | 40.0 | 53.5 |
  | 4522186 Westcoast / Very | 54 | 158.1 | 100.3 | 5.4 | 8.9 | 43.5 |
  | 2062094 Exertis (UK) Ltd | 46 | 148.7 | 90.5 | 4.5 | 12.9 | 40.8 |
  | 9037215 Combined Independent (Holdings) | 41 | 148.3 | 55.4 | 4.5 | 51.7 | 36.7 |
  | 2015540 Sky UK Ltd | 27 | 145.6 | 108.4 | 5.0 | 12.0 | 20.2 |
  | 2430099 Exertis / Very | 23 | 140.2 | 90.3 | 5.2 | 6.5 | 38.2 |
  | 2015472 Tesco Stores Limited | 21 | 139.8 | 108.7 | 10.7 | 6.8 | 13.6 |
  | 6031882 SEUK eStore - Hybris | 18 | 137.4 | 94.4 | 4.5 | 8.5 | 30.0 |
  | 2258190 Currys Retail Limited | 10 | 133.0 | 43.7 | 4.5 | 4.5 | 80.3 |

---

## Interactions summary
- Nav rail navigates between the two dashboard pages.
- Queue: Inbox/All toggle (`view`); month selector prev/next (`monthIdx`), dimmed unless All.
- Claim: month selector prev/next; Refresh (re-runs AI summary); column-header sort on by Customer.
- Chart click-to-filter: 상세 동작·datum 계약은 동작 시나리오 v3.1 **§G**를 따른다.

## Notes
- Wireframe fidelity: mid-fi. Numbers are representative sample data, not production.
- Company average for Claim L/T = 122.4 days; by Customer lists only customers above it.
