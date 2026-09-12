# Handoff: KIS Finance Web App (Desktop)

## Overview
Desktop screens for the Finance role. Finance monitors cost savings after KPO implementation: enters actual savings against KPO's approx. estimate, manages a monitoring period and status, and runs MIS reports (cost-savings, realization-rate, monitoring-status, savings-trend) with filters and export. Built against `build-spec.md` in `vasan1108/Kaizen-Management-System-` (§3 `CostRecord`, §4 state machine, §7.3 Finance reports, §10 Frontend Pages — Finance).

Same structural pattern as other back-office roles: desktop-only, table dashboard, two-column detail layout, plus a dedicated Reports tab (new for this role).

## About the Design Files
`kaizen-finance-app.html` and `browser-window.jsx` are design references; `browser-window.jsx` is a presentation-only browser-chrome wrapper. Recreate in the target stack, not by embedding this HTML.

## Fidelity
High-fidelity — final colors, type, spacing, copy.

## Design System
Same as other roles: **Modernist** base, primary `#ffd6a8` (peach), contrast `#1c1a17` (ink). Full token table in `employee.md`.

## Screens / Views

### 1. Dashboard (Desktop)
- Nav bar with two tabs: **Dashboard** (active) / **MIS Reports**.
- 4-column stat row: **Monitoring / Monitored / Overdue / Actual savings YTD** (currency total).
- Search + **plant dropdown** + **department dropdown** + priority chips (ALL/HIGH — trimmed set, Finance's queue skews toward already-approved items).
- **Table columns:** Ref ID, Employee (+ dept/line), Issue (truncated), **Approx.** (KPO's estimate), **Actual** (Finance's entry, em-dash if not yet set), **Monitoring** (status pill: outlined `MONITORING`, red-filled `OVERDUE`, ink-filled `MONITORED`).
- 960px centered content column.

### 2. Suggestion Detail — Monitoring (Desktop)
- Two-column layout (960px max). Left: Ref ID, title, meta ("Implemented by KPO ... date"), Issue + Idea text, and a condensed **Implementation stages** timeline (read-only, reused stepper pattern, no add-stage — that's KPO's action).
- Right panel:
  - "Approx. savings (KPO)" — read-only, dimmed (75% opacity) currency field, KPO's own entry.
  - "Actual savings" — editable currency field.
  - "Monitoring period" — two date fields (start/end).
  - "Monitoring status" — 2-way segmented control (MONITORING / MONITORED), selected = peach fill.
  - "SAVE" button (ink-filled, full width).

### 3. MIS Reports (Desktop)
- Nav: MIS Reports tab active.
- **Report-type tabs:** COST SAVINGS (active) / REALIZATION RATE / MONITORING STATUS / SAVINGS TREND — per build-spec §7.3.
- **Filter row:** date range, plant dropdown, department dropdown, spacer, then export buttons **CSV / XLSX / PDF** (PDF shown as the primary/ink-filled action here, others outlined — treat as equal-weight in build, this mock just varies the visual).
- **Summary stat row** (3-column): Total approx. savings / Total actual savings / Realization rate %.
- **Charts** (Cost Savings tab):
  - Bar chart: approx. (ink) vs actual (peach) savings, grouped by department — 4 department groups.
  - Line chart: monthly savings trend, two lines (approx = ink, actual = peach) — inline SVG polylines in the mock; render with a real charting lib (e.g. Recharts) in build.
- **Top 10 Contributors — [Plant name]** table: rank, Employee, Department, Suggestions (count), Actual savings — scoped to the single plant this Finance user belongs to (confirmed: one Finance team per plant).
- Below that (not separately mocked, same table pattern): a per-suggestion savings table (Ref ID, Department, Approx., Actual, Status) for the underlying Cost Savings report rows — reuse the Dashboard table styling.

## Interactions & Behavior (to implement)
- **Report tabs:** each swaps the chart(s) + table for that report type; filters (date range, plant, department) persist across tab switches within the session.
- **Export buttons:** `POST /reports/finance/:reportType/export` with `format` + current filters; logs to `ReportExportLog` per spec §7.1.
- **Top 10 Contributors:** aggregation query (submissions × actual savings) scoped to the Finance user's own plant — not cross-plant, since each plant has its own Finance team (confirmed).
- **Actual savings / monitoring period / status (detail):** `POST /suggestions/:id/cost/actual` — sets `CostRecord.actual_savings_amount`, `monitoring_period_*`, `monitoring_status`. Marking `MONITORED` should be blocked until an actual savings value is entered.
- **OVERDUE pill:** derived, not stored — flag when `monitoring_period_end` has passed and status is still `MONITORING` (per build-spec §7.3 "Monitoring Status Report").

## State Management
- Dashboard/report data fetched from API (`GET /suggestions?status=FINANCE_MONITORING...`, `GET /reports/finance/:reportType`) — not client-derived.
- Filters (plant/department/date range/priority) are query params, shared state between Dashboard and Reports where applicable.

## Files
- `kaizen-finance-app.html` — full design source (3 screens).
- `browser-window.jsx` — browser-chrome wrapper, presentation only.
- See `employee.md` for shared tokens + Sign In (reused as-is), `kpo.md` for the KPO approx. savings this role consumes.
