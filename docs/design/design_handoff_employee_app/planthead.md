# Handoff: KIS Plant Head Web App (Desktop)

## Overview
Desktop screens for the Plant Head role. Plant Head reviews suggestions already approved by a Dept Head, sees that prior review, confirms/adjusts priority, and approves/rejects with remarks before escalation to KPO (implementation). Built against `build-spec.md` in `vasan1108/Kaizen-Management-System-` (§4 state machine, §10 Frontend Pages — Plant Head).

Same structural pattern as Dept Head (see `depthead.md`): desktop-only, table dashboard, two-column detail/decision layout.

## About the Design Files
`kaizen-planthead-app.html` and `browser-window.jsx` are design references; `browser-window.jsx` is a presentation-only browser-chrome wrapper. Recreate in the target stack, not by embedding this HTML.

## Fidelity
High-fidelity — final colors, type, spacing, copy.

## Design System
Same as Employee/Dept Head: **Modernist** base, primary `#ffd6a8` (peach), contrast `#1c1a17` (ink). Full token table in `employee.md`.

## Screens / Views

### 1. Dashboard (Desktop)
- Same structure as Dept Head Dashboard, scoped to plant-wide (not single department) queue: nav bar, 4-column stat row (Pending / High priority / Approved / Rejected), search bar, **department dropdown** ("All departments" ▾ — filters by the submitting employee's department, since Plant Head sees suggestions across multiple departments), priority filter chips (ALL/HIGH/MEDIUM/LOW), and the table.
- **Table columns** (identical set to Dept Head): Ref ID, Employee (name + department/line as secondary line), Issue (single-line, truncated), Priority (swatch + label), Time in queue.
- Content constrained to 960px centered column.

### 2. Suggestion Detail — Review (Desktop)
- Same two-column layout as Dept Head detail (960px max-width, left content / right decision panel).
- **New vs. Dept Head's detail:** left column now also shows a **"Dept Head review" block** below Issue/Idea — the prior decision (Approved, priority badge), the Dept Head's remarks as a quoted line, and reviewer name + timestamp. This is read-only context for the Plant Head.
- **Right panel:** "Confirm priority" segmented control (defaults to the Dept Head's selected priority, editable), "Remarks (required)" textarea (placeholder "Add a note for the KPO…"), REJECT (outlined red) / APPROVE (ink-filled) actions.

## Interactions & Behavior (to implement)
- **Department dropdown:** single-select filter on the table, combines with search + priority filter.
- Rest identical to Dept Head: search debounce, row click → detail, priority confirm required before submit, remarks required, Approve → next stage (KPO/implementation) per build-spec §4, Reject → `REJECTED` with remarks visible to employee.
- Dept Head's prior review block is pulled from the same `SuggestionReview` record set — render whichever review stage(s) precede the current one.

## State Management
Same pattern as Dept Head — queue fetched via API scoped to plant + `status=PH_REVIEW`; decision draft (priority + remarks) local until submit; department filter is a query param.

## Files
- `kaizen-planthead-app.html` — full design source (2 screens).
- `browser-window.jsx` — browser-chrome wrapper, presentation only.
- See `employee.md` for shared tokens + Sign In (reused as-is) and `depthead.md` for the Dept Head review this role consumes.
